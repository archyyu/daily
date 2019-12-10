最近寻思做一个语音feed系统，难点呢，其实也就是在android和ios音频互通上！忙了一个星期解决了这个问题，所以就和大家分享下！

　　先说下整体的设计方案：

　　服务器：php

　　数据库：redis

　　协议：http + json

　　客户端 : android(java) + ios(oc)

　　在音频的格式选择问题上，犹豫了很久，这里其实有N个方案的，不过对于我这种非多媒体开发者来讲，还是选择一个最直接最能解决问题的就可以了！起初选择的是amr，android一切都ok，但是在ios上，wav格式的音频文件解析成amr格式的文件，或者amr格式的音频文件解码成wav格式的文件是总是出错，用的是github上的libcoreamr库，不明所以！当然如果谁解决了还是可以跟我聊下！

　　最终选择了mp3的格式！缺点就是生成的音频文件比amr文件要大！优点就是我解决了！哈哈

　　下面干货代码:首先是android 部分，相对比较简单，就是通过mic录制成mp3文件，然后上传服务器！

　　录音部分：其实就是对于MediaRecorder的使用，
```
                    recorder = new MediaRecorder();
                    recorder.setAudioSource(MediaRecorder.AudioSource.MIC);  
                    recorder.setOutputFormat(MediaRecorder.OutputFormat.MPEG_4);  
                    recorder.setAudioEncoder(MediaRecorder.AudioEncoder.AAC);  
                    recorder.setOutputFile(fileName);
                    try 
                    {  
                        recorder.prepare();  
                    } 
                    catch (IOException e) 
                    {  
                         
                    }  
                    recorder.start();
                    Toast.makeText(AddActivity.this, "开始录音", Toast.LENGTH_SHORT).show();
```
上传服务器，用的是AsyncHttp库，非常方便，这里注意的是对音频文件的上传格式问题，最好还是用文件上传的格式，php也用$_FILES[""]去提取，不要用大家平时长传图片的形式，将文件转换成byte[]，再转换成String，再给到php，php再保存文件到服务器，这样极易破坏文件的格式！注意！
```
RequestParams params = new RequestParams();
                params.put("uid", SingleManager.user.getUid());
                params.put("textContent", content.getEditableText().toString());
                try 
                {
                    params.put("radioContent", new File(fileName) );
                }
                catch (FileNotFoundException e1)
                {
                    e1.printStackTrace();
                }
                
                AsyncHttpRequestClient.post(HttpUtil.postUrl, params ,new JSONObjectResponseHandler(AddActivity.this)
                {
                    public void onJsonOk(JSONObject response)
                    {
                        try 
                        {
                        } 
                        catch (Exception e) 
                        {
                            
                        }
                        
                        Toast.makeText( AddActivity.this, "发表成功", Toast.LENGTH_SHORT).show();
                        finish();
                    } 
                    
                }); 
            }
```

如果问题到这里就简单了，后面经过测试发现，这种格式的mp3虽然可以播放，但是支持的效果很差，在第二个版本中果断放弃这种形式，而是采用一种主动编码成mp3格式的方式，使用的是github上的第三方开源库，地址：  https://github.com/yhirano/Mp3VoiceRecorderSampleForAndroid

 

　　再就是ios部分了，这里先给出一个wav转mp3的库地址，https://github.com/rpplusplus/iOSMp3Recorder，其实就是AVAudioRecorder + lame！

　　录制部分：对于AVAudioRecorder的使用和一些设置信息：
```
NSString *dir = [NSHomeDirectory() stringByAppendingPathComponent:@"documents"];
    audioFilePath = [NSString stringWithFormat:@"%@/testAudio.caf",dir];
    audioFileUrl = [NSURL fileURLWithPath:audioFilePath];
    
    mp3FilePath = [NSString stringWithFormat:@"%@/mp3Audio.mp3",dir];
    mp3FileUrl = [NSURL fileURLWithPath:mp3FilePath];
    
    
    NSMutableDictionary *recordSettings = [[NSMutableDictionary alloc] init];
    [recordSettings setValue :[NSNumber numberWithInt:kAudioFormatLinearPCM] forKey: AVFormatIDKey];
    [recordSettings setValue :[NSNumber numberWithFloat:11025.0] forKey: AVSampleRateKey];//44100.0
    [recordSettings setValue :[NSNumber numberWithInt:2] forKey: AVNumberOfChannelsKey];
    [recordSettings setValue :[NSNumber numberWithInt:16] forKey: AVLinearPCMBitDepthKey];
    [recordSettings setValue :[NSNumber numberWithInt:AVAudioQualityLow] forKey:AVEncoderAudioQualityKey];
    
    
    recorder = [[AVAudioRecorder alloc] initWithURL:audioFileUrl settings:recordSettings error:nil];
    recorder.delegate = self;
    [recorder record];
```
转码部分:在录制结束之后，要对录制文件进行转码，转换成mp3格式
```
@try {
        int read, write;
        
        FILE *pcm = fopen([audioFilePath cStringUsingEncoding:1], "rb");  //source 被转换的音频文件位置
        fseek(pcm, 4*1024, SEEK_CUR);                                   //skip file header
        FILE *mp3 = fopen([mp3FilePath cStringUsingEncoding:1], "wb");  //output 输出生成的Mp3文件位置
        
        const int PCM_SIZE = 8192;
        const int MP3_SIZE = 8192;
        short int pcm_buffer[PCM_SIZE*2];
        unsigned char mp3_buffer[MP3_SIZE];
        
        lame_t lame = lame_init();
        lame_set_in_samplerate(lame, 11025.0);
        lame_set_VBR(lame, vbr_default);
        lame_init_params(lame);
        
        do {
            read = fread(pcm_buffer, 2*sizeof(short int), PCM_SIZE, pcm);
            if (read == 0)
                write = lame_encode_flush(lame, mp3_buffer, MP3_SIZE);
            else
                write = lame_encode_buffer_interleaved(lame, pcm_buffer, read, mp3_buffer, MP3_SIZE);
            
            fwrite(mp3_buffer, write, 1, mp3);
            
        } while (read != 0);
        
        lame_close(lame);
        fclose(mp3);
        fclose(pcm);
    }
    @catch (NSException *exception) {
        NSLog(@"%@",[exception description]);
    }
    @finally
    {
    }
```
最后是上传服务器部分：这里使用的AFNETWORKING，ios常用的一个异步网络库，很好用，功能和android的aysncHttp库类似，这里和android上传使用的同一个php接口，所以音频文件按照文件形式上传，而不是转换成NSData再上传，容易破坏音频格式！
```
SingleManager *single = [SingleManager shareManager];
    AFHTTPRequestOperationManager *manager = [AFHTTPRequestOperationManager manager];
    
    NSString *uid = [NSString stringWithFormat:@"%d",single.user.userId];
    NSString *textContent = self.textContent.text;
    
    NSFileManager *fileManager = [NSFileManager defaultManager];
    NSData *data = [fileManager contentsAtPath:mp3FilePath];

    NSDictionary *parameters = @{@"uid":uid,@"textContent":textContent,@"radioContent":data};
    
    [manager POST:[baseUrl stringByAppendingString:post] parameters:parameters constructingBodyWithBlock:^(id<AFMultipartFormData> formData){[formData appendPartWithFileURL:mp3FileUrl name:@"radioContent" error:nil];
     
    }
          success:^(AFHTTPRequestOperation *operation, id responseObject)
    {
         NSMutableDictionary *result = (NSMutableDictionary *)responseObject;
         NSLog(@"Result: %@",result);
         
         [self.navigationController popViewControllerAnimated:YES];
     }
          failure:^(AFHTTPRequestOperation *operation, NSError *error)
     {
         
     }];
```
最后是php部分，这里php做的工作比较简单，就是得到客户端上传的文件，然后保存成mp3文件就可以了！还是给出部分代码:
```
$body["rId"] = $rId;$base_path = "../radio/"; //接收文件目录 
$target_path = $base_path .$rId.".mp3"; 
move_uploaded_file ( $_FILES ['radioContent'] ['tmp_name'], $target_path );
outPut($body);
```

剩下的播放部分就很简单了，无论是android还是ios，只要使用系统提供的播放器，从服务器拿到mp3文件的url，就可以只接播放了，因为mp3是通用的音频格式，无论是android还是ios都不需要做什么转换！直接播放就ok！
