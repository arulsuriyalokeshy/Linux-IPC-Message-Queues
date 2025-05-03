## Linux-IPC-Message-Queues

## AIM:
To write a C program that receives a message from message queue and display them

## DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux message queues API 

### Step 3:

Execute the C Program for the desired output. 

## PROGRAM:

## C program that receives a message from message queue and display them
```

// C Program for Message Queue (Writer Process)
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/ipc.h>
#include <sys/msg.h>

// structure for message queue
struct mesg_buffer {
    long mesg_type;
    char mesg_text[100];
} message;

int main() {
    key_t key;
    int msgid;

    // Create a file required by ftok (if not already exists)
    FILE *fp = fopen("progfile", "w");
    if (fp == NULL) {
        perror("Error creating progfile");
        exit(EXIT_FAILURE);
    }
    fclose(fp);

    // Generate unique key
    key = ftok("progfile", 65);
    if (key == -1) {
        perror("ftok failed");
        exit(EXIT_FAILURE);
    }

    // Create message queue and get identifier
    msgid = msgget(key, 0666 | IPC_CREAT);
    if (msgid == -1) {
        perror("msgget failed");
        exit(EXIT_FAILURE);
    }

    message.mesg_type = 1;

    printf("Write Data: ");
    fgets(message.mesg_text, sizeof(message.mesg_text), stdin);

    // Remove newline from fgets if present
    size_t len = strlen(message.mesg_text);
    if (len > 0 && message.mesg_text[len - 1] == '\n') {
        message.mesg_text[len - 1] = '\0';
    }

    // Send message
    if (msgsnd(msgid, &message, sizeof(message.mesg_text), 0) == -1) {
        perror("msgsnd failed");
        exit(EXIT_FAILURE);
    }

    printf("Data sent is: %s\n", message.mesg_text);

    return 0;
}

```

```
// C Program for Message Queue (Reader Process)
#include <stdio.h>
#include <stdlib.h>
#include <sys/ipc.h>
#include <sys/msg.h>

// structure for message queue
struct mesg_buffer {
    long mesg_type;
    char mesg_text[100];
} message;

int main() {
    key_t key;
    int msgid;

    // ftok to generate unique key
    key = ftok("progfile", 65);
    if (key == -1) {
        perror("ftok failed");
        exit(EXIT_FAILURE);
    }

    // msgget creates a message queue and returns identifier
    msgid = msgget(key, 0666 | IPC_CREAT);
    if (msgid == -1) {
        perror("msgget failed");
        exit(EXIT_FAILURE);
    }

    // msgrcv to receive message
    if (msgrcv(msgid, &message, sizeof(message.mesg_text), 1, 0) == -1) {
        perror("msgrcv failed");
        exit(EXIT_FAILURE);
    }

    // display the message
    printf("Data Received is: %s\n", message.mesg_text);

    // to destroy the message queue
    if (msgctl(msgid, IPC_RMID, NULL) == -1) {
        perror("msgctl failed");
        exit(EXIT_FAILURE);
    }

    return 0;
}


```

## OUTPUT:

#### Writer process
![image](https://github.com/user-attachments/assets/7e13200d-08c8-4b79-b702-6e716af138f2)


#### Reader process

![image](https://github.com/user-attachments/assets/f9641a5c-3d2b-4ef1-bafd-34489891d0f0)

## RESULT:
Thus,the programs are executed successfully.
