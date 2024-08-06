#include <stdio.h> 
#include <stdlib.h> 
#include <string.h> 
#include <unistd.h> 
#include <limits.h> 
#include <errno.h> 
 
#define MAX_DIRS 100 
 
char dir_stack[MAX_DIRS][PATH_MAX]; 
int stack_top = -1; 
 
void pushd(const char *dir) { 
    char cwd[PATH_MAX]; 
    if (getcwd(cwd, sizeof(cwd)) == NULL) { 
        perror("getcwd"); 
        return; 
    } 
     
    if (stack_top >= MAX_DIRS - 1) { 
        fprintf(stderr, "pushd: directory stack full\n"); 
        return; 
    } 
     
    if (chdir(dir) != 0) { 
        perror("pushd"); 
        return; 
    } 
     
    stack_top++; 
    strncpy(dir_stack[stack_top], cwd, PATH_MAX); 
     
    printf("%s\n", dir); 
} 
 
void popd() { 
    if (stack_top < 0) { 
        fprintf(stderr, "popd: directory stack empty\n"); 
        return; 
    } 
     
    if (chdir(dir_stack[stack_top]) != 0) { 
        perror("popd"); 
        return; 
    } 
     
    printf("%s\n", dir_stack[stack_top]); 
    stack_top--; 
} 
 
void dirs() { 
    char cwd[PATH_MAX]; 
    if (getcwd(cwd, sizeof(cwd)) == NULL) { 
        perror("getcwd"); 
        return; 
    } 
     
    printf("%s", cwd); 
    for (int i = stack_top; i >= 0; i--) { 
        printf(" %s", dir_stack[i]); 
    } 
    printf("\n"); 
} 
 
int main(int argc, char *argv[]) { 
    if (argc < 2) { 
        fprintf(stderr, "Usage: %s <command> [directory]\n", argv[0]); 
        return 1; 
    } 
 
    if (strcmp(argv[1], "pushd") == 0) { 
        if (argc != 3) { 
            fprintf(stderr, "Usage: %s pushd <directory>\n", argv[0]); 
            return 1; 
        } 
        pushd(argv[2]); 
    } else if (strcmp(argv[1], "popd") == 0) { 
        if (argc != 2) { 
            fprintf(stderr, "Usage: %s popd\n", argv[0]); 
            return 1; 
        } 
        popd(); 
    } else if (strcmp(argv[1], "dirs") == 0) { 
        if (argc != 2) { 
            fprintf(stderr, "Usage: %s dirs\n", argv[0]); 
            return 1; 
        } 
        dirs(); 
    } else { 
        fprintf(stderr, "Unknown command: %s\n", argv[1]); 
        return 1; 
    } 
 
    return 0; 
}
