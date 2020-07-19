when i developed the computer games,i usually used the locker to control the threads's access to the resource.As most the game server is single process,
so i can tackle those problems very smoothly.

However,i am a web developer now,instead of manipulating the memory,most of my time,i interact with the mysql.At first glance,most of people think 
transcation in mysql is similar to the lock in java,however this is not the truth.

i notice a bug in our project about 1 month ago,if it is for the recorded log,i probably cannot believe my eyes.

i will describe the problem simply.

There are two operations in our product,the first is logining process,the second is charging process.because our product is used for the chain netcafe,
so everyone's data is shared between different shops or netcafe.where disigning the system,i use chain_member to store the shared data,but i use the 
netbar_member to store the data in shop or netcafe.

when user logins the product,i have to copy the data from the table of chain_member to the table of netbar_member.
when user charge money,i have to check if the user is logined in the product or not.if logined,i will add money to the netbar_member,if not,i will add money
to the chain_member.

autually both processes are transcations.

at first i donnot think there is any problem until some errors happen.

When the two processes happen simultaneously. the added money is usually rewriteed by the login process.

but i know i have to find a solution to cope it.

since our team uses spring cloud in our product,these two processes may happen in different process,so the locker must be distributed.Maybe the redis can help
me to solve the problem.

after about two days,i write the first of version of the distributed locker.

package com.shic.service;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

@Service
@Slf4j
public class DistributedLocker {

    @Resource
    private RedisService redisService;

    public boolean lock(String lockKey,String requestId,Integer expireTime){
        boolean result = this.redisService.setIfAbsent(lockKey,requestId,expireTime);
        log.info("lock key:" + lockKey + ",requestid:"+ requestId + ",result:" + result);
        return result;
    }

    public boolean spinLock(String lockKey,String requestId,Integer expireTime){

        int count = 5;
        while(this.redisService.setIfAbsent(lockKey,requestId,expireTime) == false){
            if(count <= 0){
                return false;
            }
            try {
                Thread.sleep(200);
            }
            catch (Exception ex){
                log.error("err",ex);
            }
            count --;
        }
        return true;
    }


    public void release(String lockKey,String requestId){

        log.info("release key:" + lockKey + ",requestId:" + requestId);
        String value = this.redisService.get(lockKey);
        if(requestId.equals(value)){
            this.redisService.del(lockKey);
        }

    }

}


i wish the code above can help you 
