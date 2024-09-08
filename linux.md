only the ones with some kind of prog

E4-2
Rewrite the program in Q1 using vfork() and write the output


#include <stdio.h>
#include <unistd.h>
int main() {
    int a = 5, b = 10, pid;

    printf("Before vfork a=%d b=%d\n", a, b);

    pid = vfork();
    
    if (pid == 0) {
        // Child process
        a = a + 1;
        b = b + 1;
        printf("In child a=%d b=%d\n", a, b);
        _exit(0); // Use _exit() to exit child process, avoiding flushing stdio buffers
    } else {
        // Parent process
        sleep(1); // Give child some time to execute
        a = a - 1;
        b = b - 1;
        printf("In parent a=%d b=%d\n", a, b);
    }

    return 0;
}


E4-4
Complete the following program as described below :
The child process calculates the sum of odd numbers and the parent process calculate the sum of even numbers child
process to finish.
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    int pid, status;

    pid = fork();  // Create a child process

    if (pid < 0) {
        // Error occurred
        perror("fork");
        return 1;
    }

    if (pid == 0) {
        // Child process
        int odd_sum = 0;
        for (int i = 1; i <= 10; i += 2) {
            odd_sum += i;
        }
        printf("Child process: Sum of odd numbers = %d\n", odd_sum);
        _exit(0);  // Exit the child process
    } else {
        // Parent process
        wait(NULL);  // Wait for the child process to finish

        int even_sum = 0;
        for (int i = 2; i <= 10; i += 2) {
            even_sum += i;
        }
        printf("Parent process: Sum of even numbers = %d\n", even_sum);
    }

    return 0;
}


E4-6
Write a program to print the Child process ID and Parent process ID in both Child and Parent processes
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main() {
    pid_t pid, child_pid;

    pid = fork();  // Create a child process

    if (pid < 0) {
        // Fork failed
        perror("fork");
        return 1;
    }

    if (pid == 0) {
        // Child process
        child_pid = getpid();  // Get the PID of the child process
        pid_t parent_pid = getppid();  // Get the PID of the parent process
        printf("In child process: Child PID = %d, Parent PID = %d\n", child_pid, parent_pid);
    } else {
        // Parent process
        child_pid = pid;  // The PID of the child process
        pid_t parent_pid = getpid();  // Get the PID of the parent process
        wait(NULL);  // Wait for the child process to finish
        printf("In parent process: Child PID = %d, Parent PID = %d\n", child_pid, parent_pid);
    }

    return 0;
}

E5-1
create 3 threads first one to find the sum of odd numbers second one to find the sum of even numbers and third one to find the sum of natural numbers... this program also displays the list of odd/even numbers

#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>

#define N 10 // Define the range of numbers

// Function prototypes
void* sum_odd(void* arg);
void* sum_even(void* arg);
void* sum_natural(void* arg);

// Structure to hold arguments for threads
typedef struct {
    int limit;
} ThreadData;

int main() {
    pthread_t thread1, thread2, thread3;
    ThreadData data = {N}; // Initialize data with range limit N
    
    // Create threads
    if (pthread_create(&thread1, NULL, sum_odd, &data) != 0) {
        perror("Failed to create thread1");
        exit(EXIT_FAILURE);
    }
    
    if (pthread_create(&thread2, NULL, sum_even, &data) != 0) {
        perror("Failed to create thread2");
        exit(EXIT_FAILURE);
    }
    
    if (pthread_create(&thread3, NULL, sum_natural, &data) != 0) {
        perror("Failed to create thread3");
        exit(EXIT_FAILURE);
    }
    
    // Wait for threads to finish
    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);
    pthread_join(thread3, NULL);
    
    return 0;
}

// Function to calculate and print the sum of odd numbers
void* sum_odd(void* arg) {
    ThreadData* data = (ThreadData*)arg;
    int sum = 0;
    printf("Odd numbers: ");
    
    for (int i = 1; i <= data->limit; i += 2) {
        printf("%d ", i);
        sum += i;
    }
    
    printf("\nSum of odd numbers = %d\n", sum);
    return NULL;
}

// Function to calculate and print the sum of even numbers
void* sum_even(void* arg) {
    ThreadData* data = (ThreadData*)arg;
    int sum = 0;
    printf("Even numbers: ");
    
    for (int i = 2; i <= data->limit; i += 2) {
        printf("%d ", i);
        sum += i;
    }
    
    printf("\nSum of even numbers = %d\n", sum);
    return NULL;
}

// Function to calculate and print the sum of natural numbers
void* sum_natural(void* arg) {
    ThreadData* data = (ThreadData*)arg;
    int sum = 0;
    printf("Natural numbers: ");
    
    for (int i = 1; i <= data->limit; i++) {
        printf("%d ", i);
        sum += i;
    }
    
    printf("\nSum of natural numbers = %d\n", sum);
    return NULL;
}

E6-1
Execute and write the output of the following program for mutual exclusion using system V semaphore 
#include<sys/ipc.h> #include<sys/sem.h> int main() 
{ 
int pid, semid,val; 
struct sembuf sop; 
semid=semget((key_t) 6,1,IPC_CREAT|0666); 
pid=fork(); 
sop.sem num=0; 
sop.sem_op=0; 
sop.sem flg=0; 
if (pid!=0) 
{ 
sleep (1); 
printf("The Parent waits for WAIT signal\n"); 
semop (semid, & sop, 1); 
printf("The Parent WAKED UP & doing her job\n"); sleep(10); 
printf("Parent Over\n"); 
else 
{ 
printf("The Child sets WAIT signal & doing her job\n"); semctl (semid, 0, SETVAL, 1); 
sleep(10); 
printf("The Child sets WAKE signal & finished her job\n"); semctl (semid, 0, SETVAL, 0); 
printf("Child Over\n"); 
} 
return 0; 
} 


E6-2
Program creates two threads: one to increment the value of a shared variable and second to decrement the value of the shared variable. Both the threads make use of semaphore variable so that only one of the threads is executing in its critical section. Execute and write the output. 
#include<pthread.h> 
#include<stdio.h> 
#include<semaphore.h> 
#include<unistd.h> 
void *fun1(); 
void *fun2(); 
int shared=1; //shared variable 
sem_t s; //semaphore variable 
int main() 
{ 
sem init(&s, 0,1); //initialize semaphore variable - 1st argument is //address of variable, 2nd is number of processes sharing semaphore, //3rd argument is the initial value of semaphore variable 
pthread_t thread1, thread2; 
pthread create (&thread1, NULL, funl, NULL); 
sleep(1); 
pthread create (&thread2, NULL, fun2, NULL); 
pthread_join (thread1, NULL); 
pthread_join(thread2,NULL); 
printf("Final value of shared is %d\n",shared); //prints the last //updated value of shared variable 
} 
void *fun1() 
{ 
int x; 
sem wait(&s); //executes wait operation on s x=shared; //thread1 reads value of shared variable printf("Threadl reads the value as %d\n",x); 
x++; //thread1 increments its value 
printf("Local updation by Threadl: %d\n",x); 
sleep(1); //thread1 is preempted by thread 2 
shared=x; //thread one updates the value of shared variable printf("Value of shared variable updated by Threadl is: 
%d\n", shared); 
} 
sem_post(&s); 
void *fun2 () 
{ 
int y; 
sem wait(&s); 
y=shared; //thread2 reads value of shared variable printf("Thread2 reads the value as %d\n",y); 
y--; //thread2 increments its value 
printf("Local updation by Thread2: %d\n", y); 
sleep (1); //thread2 is preempted by thread 1 
shared=y; //thread2 updates the value of shared variable printf("Value of shared variable updated by Thread2 is: %d\n", shared); 
} 
sem_post(&S);


E6-3
Reader writer program

#include <pthread.h>
#include <semaphore.h>
#include <stdio.h>

sem_t wrt;
pthread_mutex_t mutex;
int cnt = 1;
int numreader = 0;

void *writer(void *wno) {
    sem_wait(&wrt);
    cnt = cnt * 2;
    printf("Writer %d modified cnt to %d\n", (*((int*)wno)), cnt);
    sem_post(&wrt);
}

void *reader(void *rno) {
    pthread_mutex_lock(&mutex);
    numreader++;
    if (numreader == 1) {
        sem_wait(&wrt);
    }
    pthread_mutex_unlock(&mutex);

    // Reading Section

    pthread_mutex_lock(&mutex);
    numreader--;
    if (numreader == 0) {
        sem_post(&wrt);
    }
    pthread_mutex_unlock(&mutex);
    numreader--;
    if(numreader==0){
        sem_post(&wrt);
    }
    pthread_mutex_unlock
}
int main(){
    pthead_t read[10], write[5];
    pthread_mutex_init(&mutex, NULL);
    sem_init(&wrt, 0,1);

    int a[10] = {1,2,3,4,5,6,7,8,9,10};
    for(int i=0;i<10;i++){
        pthread_create(&read[i], NULL, (void *)reader, (void *)&a[i]);
    }
    for (int i = 0; i < 5; i++) { 
        pthread_create (&write[i], NULL, (void *) writer, (void *) &a[i]); 
        }

    for (int i = 0; i < 10; i++){
        pthread_join(read[i], NULL);
    }
    for (int i = 0; i < 5; i++){
        pthread_join(write[i], NULL);
    }
    return 0;

}
