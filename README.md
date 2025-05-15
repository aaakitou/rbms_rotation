#include "mbed.h"
#include "rbms.h" 

#define TOTAL_MOTOR 4

UnbufferedSerial pc(USBTX, USBRX, 921600);
//CAN can(PA_11, PA_12);//303
CAN can(PD_0,PD_1);//767
rbms m2006(can,0,TOTAL_MOTOR);
Thread thread1, thread_can;
EventQueue queue(32 * EVENTS_EVENT_SIZE);
void onReceive();
void motor_deg_control(); 
void can_receive();

CANMessage msg;
int set_deg[TOTAL_MOTOR]={0}; // 回したい角度（ギアのかんだあとの方）
int motor[TOTAL_MOTOR]={0};
// Timer a;
// int co=0;
// float t;
int main(){
    char key;
    //a.start();
    //thread_can.start(can_receive); // スレッドの開始 
    thread1.start(motor_deg_control);
    thread_can.start(callback(&queue, &EventQueue::dispatch_forever));
    can.attach(onReceive, CAN::RxIrq); // 受信割り込みハンドラを設定
    
    while(true){
        if(pc.readable()){
            pc.read(&key, 1);
            switch(key){
                case 't':
                    printf("'%c' pushed  ", key);
                    set_deg[0] =  1080;
                    break;
                case 'g':
                    printf("'%c' pushed  ", key);
                    set_deg[0] =  3600;
                    break;
                case 'b':
                    printf("'%c' pushed  ", key);
                    set_deg[0] = 1;
                    break;
                case 'i':
                    printf("'%c' pushed  ", key);
                    set_deg[0]=120;
                    break;
                case 'o':
                    printf("'%c' pushed  ", key);
                    set_deg[0]=-120;
                    break;
                case 'y':
                    set_deg[0]=8;
                    break;
            }
        }
        // t=a.read();
        // if(t>=1.0){
        // printf("%d\r\n",co);
        // co=0;
        // a.reset();
        // }
        m2006.rbms_send(motor);
        ThisThread::sleep_for(5ms);
    }
}

void motor_deg_control(){
    m2006.deg_control(set_deg, motor); // マルチスレッドで書く必要がある
}

void can_receive(){
    m2006.can_read(); // canは常に受信させておく
    //co+=1;
}

void onReceive() {
    queue.call(can_receive); // EventQueueにCANreceive関数を登録
}
