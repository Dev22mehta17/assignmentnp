# assignmentnp
assi-2
//server1.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
int main() {
 int sockfd, newsockfd, portno;
 socklen_t clilen;
 char buffer[256];
 struct sockaddr_in serv_addr, cli_addr;
 sockfd = socket(AF_INET, SOCK_STREAM, 0);
 if (sockfd < 0) {
 perror("Error opening socket");
 exit(1);
 }
 bzero((char *) &serv_addr, sizeof(serv_addr));
 portno = 12345;
 serv_addr.sin_family = AF_INET;
 serv_addr.sin_addr.s_addr = INADDR_ANY;
 serv_addr.sin_port = htons(portno);
 if (bind(sockfd, (struct sockaddr *) &serv_addr, sizeof(serv_addr)) < 0) {
 perror("Error on binding");
 exit(1);
 }
 listen(sockfd, 5);
 clilen = sizeof(cli_addr);
 newsockfd = accept(sockfd, (struct sockaddr *) &cli_addr, &clilen);
 if (newsockfd < 0) {
 perror("Error on accept");
 exit(1);
 }
 bzero(buffer, 256);
 int n = read(newsockfd, buffer, 255);
 if (n < 0) {
 perror("Error reading from socket");
 exit(1);
 }
 printf("Message from client: %s\n", buffer);
 n = write(newsockfd, buffer, strlen(buffer));
 if (n < 0) {
 perror("Error writing to socket");
 exit(1);
 }
 close(newsockfd);
 close(sockfd);
 return 0;
}
// client1.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <netdb.h>
int main() {
 int sockfd, portno, n;
 struct sockaddr_in serv_addr;
 struct hostent *server;
 char buffer[256];
 portno = 12345;
 sockfd = socket(AF_INET, SOCK_STREAM, 0);
 if (sockfd < 0) {
 perror("Error opening socket");
 exit(1);
 }
 server = gethostbyname("127.0.0.1");
 if (server == NULL) {
 fprintf(stderr, "Error, no such host\n");
 exit(0);
 }
 bzero((char *) &serv_addr, sizeof(serv_addr));
 serv_addr.sin_family = AF_INET;
 bcopy((char *)server->h_addr,
 (char *)&serv_addr.sin_addr.s_addr,
 server->h_length);
 serv_addr.sin_port = htons(portno);
 if (connect(sockfd, (struct sockaddr *) &serv_addr, sizeof(serv_addr)) < 0) {
 perror("Error connecting");
 exit(1);
 }
 printf("Enter a message: ");
 bzero(buffer, 256);
 fgets(buffer, 255, stdin);
 n = write(sockfd, buffer, strlen(buffer));
 if (n < 0) {
 perror("Error writing to socket");
 exit(1);
 }
 bzero(buffer, 256);
 n = read(sockfd, buffer, 255);
 if (n < 0) {
 perror("Error reading from socket");
 exit(1);
 }
 printf("Message from server: %s\n", buffer);
 close(sockfd);
 return 0;
}
ass-3
//echo server
#include<stdio.h>
#include<netinet/in.h>
#include<sys/types.h>
#include<sys/socket.h>
#include<netdb.h>
//UDP Echo Server
#include<string.h>
#include<stdlib.h>

int main()
{
int sockfd,n,clen;
struct sockaddr_in servaddr,cli;
char buff[80];


sockfd=socket(AF_INET,SOCK_DGRAM,0);

bzero(&servaddr,sizeof(servaddr));
servaddr.sin_family=AF_INET;
servaddr.sin_addr.s_addr=htonl(INADDR_ANY);
servaddr.sin_port=htons(43454);

bind(sockfd,(struct sockaddr *)&servaddr,sizeof(servaddr)

clen=sizeof(cli);

while(1)
{
bzero(buff,80);
recvfrom(sockfd,buff,sizeof(buff),0,(struct sockaddr *)&cli,&clen);
printf("\nUDP Echo Back: %s ",buff);	
sendto(sockfd,buff,strlen(buff),0,(struct sockaddr *)&cli,clen);

if(strncmp("exit",buff,4)==0)
{
printf("Client Exit...\n");
break;
}

}
close(sockfd);
}
//udpechoclient
#include<sys/socket.h>
#include<netdb.h>
#include<string.h>
#include<stdlib.h>
#include<stdio.h>

int main()
{
char buff[80];
int sockfd,len,n;
len=sizeof(servaddr);
struct sockaddr_in servaddr;

sockfd=socket(AF_INET,SOCK_DGRAM,0);


bzero(&servaddr,sizeof(len));
servaddr.sin_family=AF_INET;
servaddr.sin_addr.s_addr=inet_addr("127.0.0.1");
servaddr.sin_port=htons(43454);



printf("\nEnter string : ");
n=0;
while((buff[n++]=getchar())!='\n');

sendto(sockfd,buff,sizeof(buff),0,(struct sockaddr *)&servaddr,len);
bzero(buff,sizeof(buff));
recvfrom(sockfd,buff,sizeof(buff),0,(struct sockaddr *)&servaddr,&len);
printf("From Server : %s\n",buff);

close(sockfd);
}
Ass-4
//tch chat server
#include <stdio.h>
#include <signal.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <netdb.h>
#include <netinet/in.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <sys/types.h>
#define MAX 80
#define PORT 8080
#define SA struct sockaddr
// Function designed for chat between client and server.
void func(int sockfd, short cport)
{
char buff[MAX];
int n;

// infinite loop for chat
for (;;) {
    bzero(buff, MAX); // set buff with null values
// read the message from client and copy it in buffer
    read(sockfd, buff, sizeof(buff));
	//recvfrom
// print buffer which contains the client contents
    printf("From client(%d): %s\nTo client(%d) : ",cport, buff,cport);
    bzero(buff, MAX);
    n = 0;
 // copy server message in the buffer
    while ((buff[n++] = getchar()) != '\n')
      ;
// and send that buffer to client
    write(sockfd, buff, sizeof(buff));
	//sendto
// if msg contains "Exit" then server exit and chat ended.
   if (strncmp("exit", buff, 4) == 0) {
    printf("Server Exit...\n");
   break;
   }
}
}



// Driver function
int main()
{
int sockfd, connfd, len;
struct sockaddr_in servaddr, cli;
pid_t pid;
//void sig_chld(int);
// socket create and verification
sockfd = socket(AF_INET, SOCK_STREAM, 0);

if (sockfd == -1) {
printf("socket creation failed...\n");
exit(0);
}
else
printf("Socket successfully created..\n");



bzero(&servaddr, sizeof(servaddr));
// assign IP, PORT
servaddr.sin_family = AF_INET;
servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
servaddr.sin_port = htons(PORT);



// Binding newly created socket to given IP and verification
if ((bind(sockfd, (SA*)&servaddr, sizeof(servaddr))) != 0) {
printf("socket bind failed...\n");
exit(0);
}
else
printf("Socket successfully binded..\n");
// Now server is ready to listen and verification
if ((listen(sockfd, 5)) != 0) {
printf("Listen failed...\n");
exit(0);
}
else
printf("Server listening..\n");
len = sizeof(cli);


while(1){
// Accept the data packet from client and verification
connfd = accept(sockfd, (SA*)&cli, &len);
if (connfd < 0) {
printf("server acccept failed...\n");
exit(0);
}
else{
printf("server acccept the client...\n");
short c_port= ntohs(cli.sin_port);
// Function for chatting between client and server	
func(connfd,c_port);
  }//end else
}//end while(1)

close(sockfd);// After chatting close the socket
}
tcp client chat
#include <netdb.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#define MAX 80
#define PORT 8080
#define SA struct sockaddr
void func(int sockfd)
{
char buff[MAX];
int n;
for (;;) {
bzero(buff, sizeof(buff));
printf("Enter the string : ");
n = 0;
while ((buff[n++] = getchar()) != '\n')
;
write(sockfd, buff, sizeof(buff));//send the message to server
bzero(buff, sizeof(buff)); //nullifies the buffer content
read(sockfd, buff, sizeof(buff)); //reading the message recived from server
printf("From Server : %s", buff);
if ((strncmp(buff, "exit", 4)) == 0) {
printf("Client Exit...\n");
close(sockfd);
break;
}
}
}
int main()
{
int sockfd, connfd;
struct sockaddr_in servaddr, cli;
// socket create and varification
sockfd = socket(AF_INET, SOCK_STREAM, 0);
if (sockfd == -1) {
printf("socket creation failed...\n");
exit(0);
}
else
printf("Socket successfully created..\n");
bzero(&servaddr, sizeof(servaddr));
// assign IP, PORT
servaddr.sin_family = AF_INET;
servaddr.sin_addr.s_addr = inet_addr("127.0.0.1");
servaddr.sin_port = htons(PORT);
// connect the client socket to server socket
if (connect(sockfd, (SA*)&servaddr, sizeof(servaddr)) != 0) {
printf("connection with the server failed...\n");
exit(0);
}
else
printf("connected to the server..\n");
// function for chat
func(sockfd);
// close the socket
close(sockfd);
}
Ass-5
udp echo client1
#include<stdio.h>
#include<netinet/in.h>
#include<sys/types.h>
#include<sys/socket.h>
#include<netdb.h>
//UDP Echo Server
#include<string.h>
#include<stdlib.h>

int main()
{
int sockfd,n,clen;
struct sockaddr_in servaddr,cli;
char buff[80];


sockfd=socket(AF_INET,SOCK_DGRAM,0);
if(sockfd==-1)
{
printf("socket creation failed...\n");
exit(0);
}
else
printf("Socket successfully created..\n");
bzero(&servaddr,sizeof(servaddr));
servaddr.sin_family=AF_INET;
servaddr.sin_addr.s_addr=htonl(INADDR_ANY);
servaddr.sin_port=htons(43454);
if((bind(sockfd,(struct sockaddr *)&servaddr,sizeof(servaddr)))!=0)
{
printf("socket bind failed...\n");
exit(0);
}
else
printf("Socket successfully binded..\n");
clen=sizeof(cli);
while(1)
{
bzero(buff,80);
recvfrom(sockfd,buff,sizeof(buff),0,(struct sockaddr *)&cli,&clen);
printf("\nUDP Echo Back: %s ",buff);	
sendto(sockfd,buff,strlen(buff),0,(struct sockaddr *)&cli,clen);
if(strncmp("exit",buff,4)==0)
{
printf("Client Exit...\n");
break;
}
}
close(sockfd);
}


udp echo client2

#include<sys/socket.h>
#include<netdb.h>
#include<string.h>
#include<stdlib.h>
#include<stdio.h>

int main()
{
char buff[80];
int sockfd,len,n;
struct sockaddr_in servaddr;
sockfd=socket(AF_INET,SOCK_DGRAM,0);
if(sockfd==-1)
{
printf("socket creation failed...\n");
exit(0);
}
else
printf("Socket successfully created..\n");
bzero(&servaddr,sizeof(len));
servaddr.sin_family=AF_INET;
servaddr.sin_addr.s_addr=inet_addr("127.0.0.1");
servaddr.sin_port=htons(43454);
len=sizeof(servaddr);


printf("\nEnter string : ");
n=0;
while((buff[n++]=getchar())!='\n')
	;


sendto(sockfd,buff,sizeof(buff),0,(struct sockaddr *)&servaddr,len);
bzero(buff,sizeof(buff));
recvfrom(sockfd,buff,sizeof(buff),0,(struct sockaddr *)&servaddr,&len);
printf("From Server : %s\n",buff);

close(sockfd);
}
ass-6
broadcast
Sending process:
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <string.h>

#define  err_log(log) do{perror(log); exit(1);}while(0)

#define  N 128

int main(int argc, const char *argv[])
{

    int sockfd;
    struct sockaddr_in broadcastaddr;
    char buf[N] = {0};

    if((sockfd = socket(AF_INET, SOCK_DGRAM, 0)) < 0)
    {
        err_log("fail to socket");
    }

    broadcastaddr.sin_family = AF_INET;
    broadcastaddr.sin_addr.s_addr = inet_addr("192.168.1.255");        //Broadcast address
    broadcastaddr.sin_port = htons(10000);

    int optval = 1;

    if(setsockopt(sockfd, SOL_SOCKET, SO_BROADCAST, &optval, sizeof(int)) < 0)
    {
        err_log("fail to setsockopt");
    }

    while(1)
    {
        printf("Input > ");
        fgets(buf, N, stdin);
        if(sendto(sockfd,buf, N, 0, (struct sockaddr *)&broadcastaddr, sizeof(broadcastaddr)) < 0)
        {
            err_log("fail to sendto");
        }
    }
    return 0;
}



Receiving process:
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <string.h>

#define  err_log(log) do{perror(log); exit(1);}while(0)
#define  N  128

int main(int argc, const char *argv[])
{

    int sockfd;
    char buf[N];
    struct sockaddr_in broadcastaddr, srcaddr;

    if((sockfd = socket(AF_INET, SOCK_DGRAM, 0)) < 0)
    {
        err_log("fail to socket");
    }

    broadcastaddr.sin_family = AF_INET;
    broadcastaddr.sin_addr.s_addr = inet_addr("INADDR_ANY or 192.168.1.255");   
                                                                                       //Broadcast address/ INADDR_ANY

    broadcastaddr.sin_port = htons(10000);

    if(bind(sockfd, (struct sockaddr*)&broadcastaddr, sizeof(broadcastaddr)) < 0)
    {
        err_log("fail to bind");
    }

    socklen_t addrlen = sizeof(struct sockaddr);

    while(1)
    {
        if(recvfrom(sockfd,buf, N, 0, (struct sockaddr *)&srcaddr, &addrlen) < 0)
        {
            err_log("fail to sendto");
        }
        printf("buf:%s ---> %s %d\n", buf, inet_ntoa(srcaddr.sin_addr), ntohs(srcaddr.sin_port));
    }

    return 0;
}

Ass-7
Sending multicast datagrams:
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <stdio.h>
#include <stdlib.h>
struct in_addr localInterface;
struct sockaddr_in groupSock;
int sd;
int datalen;
char databuf[1024];
int main (int argc, char *argv[])
{
 /*
 * Create a datagram socket on which to send.
 */
 sd = socket(AF_INET, SOCK_DGRAM, 0);
 if (sd < 0) {
 perror("opening datagram socket");
 exit(1);
 }
 /*
 * Initialize the group sockaddr structure with a
 * group address of 225.1.1.1 and port 5555.
 */
 memset((char *) &groupSock, 0, sizeof(groupSock));
 groupSock.sin_family = AF_INET;
 groupSock.sin_addr.s_addr = inet_addr("225.1.1.1");
 groupSock.sin_port = htons(5555);
 /*
 * Disable loopback so you do not receive your own datagrams.
 */
 {
 char loopch=0;
 if (setsockopt(sd, IPPROTO_IP, IP_MULTICAST_LOOP, (char *)&loopch, sizeof(loopch)) < 0) {
 perror("setting IP_MULTICAST_LOOP:");
 close(sd);
 exit(1);
 }
 }
 /*
 * Set local interface for outbound multicast datagrams.
 * The IP address specified must be associated with a local,
 * multicast-capable interface.
 */
 localInterface.s_addr = inet_addr("9.5.1.1");
 if (setsockopt(sd, IPPROTO_IP, IP_MULTICAST_IF, (char *)&localInterface, sizeof(localInterface)) < 0) {
 perror("setting local interface");
 exit(1);
 }
 /*
 * Send a message to the multicast group specified by the
 * groupSock sockaddr structure.
 */
 datalen = 10;
 if (sendto(sd, databuf, datalen, 0,
 (struct sockaddr*)&groupSock,
 sizeof(groupSock)) < 0)
 {
 perror("sending datagram message");
 }}
Receiving multicast datagrams:
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <stdio.h>
#include <stdlib.h>
struct sockaddr_in localSock;
struct ip_mreq group;
int sd;
int datalen;
char databuf[1024];
int main (int argc, char *argv[])
{
 /*
 * Create a datagram socket on which to receive.
 */
 sd = socket(AF_INET, SOCK_DGRAM, 0);
 if (sd < 0) {
 perror("opening datagram socket");
 exit(1);
 }
 /*
 * Enable SO_REUSEADDR to allow multiple instances of this application to receive copies of the multicast 
datagrams.
 */
 {
 int reuse=1;
 if (setsockopt(sd, SOL_SOCKET, SO_REUSEADDR, (char *)&reuse, sizeof(reuse)) < 0) {
 perror("setting SO_REUSEADDR");
 close(sd);
 exit(1);
 }
 }
 /*
 * Bind to the proper port number with the IP address specified as INADDR_ANY.
 */
 memset((char *) &localSock, 0, sizeof(localSock));
 localSock.sin_family = AF_INET;
 localSock.sin_port = htons(5555);;
 localSock.sin_addr.s_addr = INADDR_ANY;
 if (bind(sd, (struct sockaddr*)&localSock, sizeof(localSock))) {
 perror("binding datagram socket");
 close(sd);
 exit(1);
 }
 /*
 * Join the multicast group 225.1.1.1 on the local 9.5.1.1
 * interface. Note that this IP_ADD_MEMBERSHIP option must be
 * called for each local interface over which the multicast
 * datagrams are to be received.
 */
 group.imr_multiaddr.s_addr = inet_addr("225.1.1.1");
 group.imr_interface.s_addr = inet_addr("9.5.1.1");
 if (setsockopt(sd, IPPROTO_IP, IP_ADD_MEMBERSHIP, (char *)&group, sizeof(group)) < 0) {
 perror("adding multicast group");
 close(sd);
 exit(1);
 }
 /*
 * Read from the socket.
 */
 datalen = sizeof(databuf);
 if (read(sd, databuf, datalen) < 0) {
 perror("reading datagram message");
 close(sd);
 exit(1);


