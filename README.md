#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include "../inc/shell.h"

#define NB_BUILTIN_FUNCTION 3

typedef enum {
    COMMAND_EXIT_SUCCESS = 0,
    COMMAND_EXIT_FAILURE = 1,
    COMMAND_SUCCESS = 2,
    COMMAND_NOT_FOUND = -1
} eCommandReturnCode;

char* readUserInput(void);
char** parseUserInput(char *userInput);

int execute(char **arguments);

int shellExit(char **arguments);
int shellEcho(char **arguments);
int shellType(char **arguments);

char *builtinCommands[] = {
    "exit",
    "echo",
    "type"
};

/* shell functions must follow the same order as in the above command table */
int (*builtinFunctions[]) (char**) = {
    &shellExit,
    &shellEcho,
    &shellType
};

#define BUFFER_SIZE 4096
#define TOKEN_BUFFER_SIZE 64

#define TOKEN_DELIM " "

int main(int argc, char *argv[]){
    eCommandReturnCode returnCode = COMMAND_SUCCESS;
    char *userInputBuffer;
    char **arguments;
    char *userInputBufferCpy;
    
    while (1) {
        setbuf(stdout, NULL);
        printf("$ ");
        
        userInputBuffer = readUserInput();
        // Make a copy before sending it to ther parser as it will be destroyed
        userInputBufferCpy = strcpy(userInputBufferCpy, userInputBuffer);
        arguments = parseUserInput(userInputBuffer);
        printf("INPUT %s\n", userInputBufferCpy);
        returnCode = execute(arguments);

        if (COMMAND_EXIT_SUCCESS == returnCode) {
            break;
        }
        else if (COMMAND_EXIT_FAILURE == returnCode) {
            break;
        }
        else if (COMMAND_NOT_FOUND == returnCode) {
            printf("%s: command not found\n", userInputBufferCpy);
        }
        // Otherwise, continue
    }

    free(userInputBuffer);
    free(arguments);

    return returnCode;
}

char* readUserInput(void) {
    // To read each charater of the user input. Set as an int for comparison with EOF
    int ch;
    int userInputIndex = 0;
    int userInputBufferSize = BUFFER_SIZE;
    char *userInputBuffer = calloc(BUFFER_SIZE, sizeof(char));

    if (NULL == userInputBuffer) {
        fprintf(stderr, "Initial Memory Allocation error for User Input buffer Failed\n");
        exit(EXIT_FAILURE);
    }

    while (1) {
        ch = getchar();

        /* End of the user input 
         * Set the last character as null character and return
         */
        if ((EOF == ch) ||('\n' == ch)) {
            userInputBuffer[userInputIndex] = '\0';
            return userInputBuffer;
        }

        userInputBuffer[userInputIndex] = ch;
        userInputIndex++;

        /* In case the buffer size is too small, reallocate */
        if (userInputIndex >= userInputBufferSize) {
            userInputBufferSize += BUFFER_SIZE;
            userInputBuffer = realloc(userInputBuffer, userInputBufferSize);
            if (NULL == userInputBuffer) {
                fprintf(stderr, "Memory Reallocation error for User Input buffer Failed\n");
                exit(EXIT_FAILURE);
            }
        }
    }
}

char** parseUserInput(char *userInput) {
    int tokenArraySize = TOKEN_BUFFER_SIZE;
    int tokentArrayIndex = 0;
    char *token;
    
    /* Array of string storing all the tokens present in userInput */
    char **tokenArray = calloc(tokenArraySize, sizeof(char*));
    if (NULL == tokenArray) {
        fprintf(stderr, "Initial Memory Allocation error for token array\n");
        exit(EXIT_FAILURE);
    }   

    token = strtok(userInput, TOKEN_DELIM);

    while (NULL != token) {
        tokenArray[tokentArrayIndex] = token;
        tokentArrayIndex++;

        if (tokentArrayIndex >= tokenArraySize) {
            tokenArraySize += TOKEN_BUFFER_SIZE;
            tokenArray = realloc(tokenArray, tokenArraySize*sizeof(char*));
            if (NULL == tokenArray) {
                fprintf(stderr, "Memory Reallocation error for token array\n");
                exit(EXIT_FAILURE);
            }           
        }
        token = strtok(NULL, TOKEN_DELIM);
    }

    tokenArray[tokentArrayIndex] = NULL;
    return tokenArray;
}

int execute(char **arguments) {
    int functionIndex = 0;

    // No command typed, considered as successful command
    if (NULL == arguments[0]) {
        return COMMAND_SUCCESS;
    }

    for ( functionIndex = 0; functionIndex < NB_BUILTIN_FUNCTION; functionIndex++)
    {
        if (0 == strcmp(builtinCommands[functionIndex], arguments[0])) {
            return (*builtinFunctions[functionIndex])(arguments);
        }
    }
    return COMMAND_NOT_FOUND;
}

int shellExit(char **arguments) {
    // Too Few arguments
    if (NULL == arguments[1]) {
        return COMMAND_NOT_FOUND;
    }
    else if (0 == strcmp("0", arguments[1])) {
        return COMMAND_EXIT_SUCCESS;
    }
    else if (0 == strcmp("1", arguments[1])) {
        return COMMAND_EXIT_FAILURE;
    }
    else {
        return COMMAND_NOT_FOUND;
    }
}
int shellEcho(char **arguments) {
    for (int i = 1; NULL != arguments[i]; i++)
    {
        printf("%s ", arguments[i]);
    }
    printf("\n");
}
int shellType(char **arguments) {

}
