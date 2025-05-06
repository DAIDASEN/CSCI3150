```c
#include <pthread.h>
#include <stdio.h>

pthread_mutex_t m;
pthread_cond_t toWorld, toHello;
pthread_t a,b;

int alternate = 1;  // 应该填入 1
void *hello(void *param){
    while(1){
        pthread_mutex_lock(&m);    /* protect alternate */
        while (alternate!=0) {     /* If not print world, then wait */
            pthread_cond_wait(&toHello, &m);
        }
        

        printf ("%s", param);
        alternate = 1 ;
        pthread_cond_signal(&toWorld);  /* wake up world */
        pthread_mutex_unlock(&m);  /* release alternate */
    }

}

void *world(void *param){
    while(1){
        pthread_mutex_lock(&m);    /* protect alternate */
        while (alternate!=1){      /* If not print hello, then wait */
            pthread_cond_wait(&toWorld, &m);
        }
        

        printf("%s", param);
        alternate = 0 ;
        pthread_cond_signal(&toHello);  /* wake up hello */
        pthread_mutex_unlock(&m);  /* release alternate */
    }

}

int main(){
    // Initialize the mutex and condition variables
    pthread_mutex_init(&m, NULL);
    pthread_cond_init(&toHello, NULL);
    pthread_cond_init(&toWorld, NULL);  // 需要添加
    

    pthread_create(&a, NULL, hello, "Hello ");
    pthread_create(&b, NULL, world, "World\n");
    
    // Wait for the threads to finish
    pthread_join(a, NULL);
    pthread_join(b, NULL);  // 实际上这两个线程都是无限循环，不会结束
    
    return 0;  // 需要添加

}
```

