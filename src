/**
 * Simple shell interface program.
 *
 * Operating System Concepts - Tenth Edition
 * Copyright John Wiley & Sons - 2018
 */

#include <stdio.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
#include <string.h>
#include <fcntl.h>
#define MAX_LINE		80 /* 80 chars per line, per command */

int main(void)
{
	FILE* infile;
	char* args[MAX_LINE / 2 + 1];	/* command line (of 80) has max of 40 arguments */
	char* args2[MAX_LINE / 2 + 1] = { NULL }; // to excute the second argument if we useed | operator
	char temp[MAX_LINE]; // to take the comman as a string then parse it in args
	char temp2[MAX_LINE]; // to make a copy of the command " temp" to use it for history
	int should_run = 1;
	char* HISTORY[MAX_LINE] = { NULL }; // keep track of the last command written to excute again if we used !! operator
	int saved_stdout; // to make the output the screen again after switching it to a file
	int saved_stdin;// to make the input the keyboard again after switching it to a file
	int check = 0; // to check on < operator
	int check2 = 0; // to check on > operator
	int check3 = 0; // to check on | operator
	int pipe_array[2]; // for piping pipe_array[0] to read and [1] is to write
	while (should_run)
	{

		check = 0; // re-enitializing it so we donot get confussed from one iteration to the next
		check2 = 0;
		check3 = 0;
		saved_stdout = dup(1); // to regain my state if i switched my output to a file instead the keyboard
		saved_stdin = dup(0); // to regain my state if i switched my input to a file instead the keyboard
		printf("osh>");
		fflush(stdout);
		scanf("%[^\n]%*c", temp); // take a string until /n including the /n so we can check on the null character and parse successfully
		printf("command %s \n", temp);
		int i = 0;
		if (temp[0] != '!' && temp[1] != '!') // not to change the history if the new command is !! operator so we can run the last command again
			strcpy(temp2, temp);
		args[i] = strtok(temp, " "); // parsing the command

		while (args[i] != NULL) // parsing the command
		{
			i++;
			args[i] = strtok(NULL, " ");


		}
		if (strcmp(args[0], "!!") != 0) // the new command is not !! then we change the history buffer
		{

			int j = 0;
			HISTORY[j] = strtok(temp2, " ");

			while (HISTORY[j] != NULL)
			{
				j++;
				HISTORY[j] = strtok(NULL, " ");


			}
		}
		else if (HISTORY[0] != NULL) // their is a command in the history buffer so we will execute it as the new command
		{
			int	 j = 0;
			args[j] = HISTORY[j];
			while (HISTORY[j] != NULL)
			{

				j++;
				args[j] = HISTORY[j];

			}
		}
		else // if history is equal to null then their is no history and exit as an error
		{
			should_run = 0;
			printf("no previous history \n");
			break;
		}

		if (strcmp(args[0], "exit") == 0) // the comman is exit then we should close the terminal
		{
			should_run = 0;
			printf("exit call \n");
			break;
		}


		if (strcmp(args[i - 1], "&") != 0) // if the last char is not & then we execute the child and the parent wait for it until it finishes
		{

			int j = 0;
			while (args[j] != NULL) // to check if we have any of > < | operands
			{
				if (strcmp(args[j], ">") == 0)
					check = j;
				else if (strcmp(args[j], "<") == 0)
					check2 = j;
				else if (strcmp(args[j], "|") == 0)
					check3 = j;
				j++;
			}

			if (check != 0) // if we have < operand then we change the output to a file
			{
				int file = open(args[check + 1], O_APPEND | O_WRONLY);
				dup2(file, 1);

			}
			else if (check2 != 0) // if we have > operand then we change the input from a file
			{
				int file = open(args[check2 + 1], O_RDONLY);
				dup2(file, 0);
			}
			else if (check3 != 0) // if we have | then we excute 2 comman using pipe
			{
				int j = check3;
				int k = 0;
				while (args[j] != NULL) // to have 2 string to pass to exec() each string is a comman one before | and one after
				{
					if (strcmp(args[j], "|") != 0) {
						args2[k] = args[j];
						k++;
					}
					args[j] = NULL;
					j++;

				}

			}
			pid_t pid;
			pid = fork();
			if (pid < 0) // we failed to create a child then we excete
			{
				printf("the child process was not created, their was an error during fork \n");
				should_run = 0;
				return 1;
			}
			else if (pid == 0) // the child process
			{
				if (check3 != 0) // if we have | operand then we should create another child
				{
					if (pipe(pipe_array) < 0) // something wrong happened while creating a pipe
					{
						should_run = 0;
						break;
					}
					else // pipe was created
					{
						pid = fork(); // making a child process for the child
						if (pid > 0) // executing the first command
						{
							dup2(pipe_array[1], 1);
							close(pipe_array[1]);
							close(pipe_array[0]);
							execvp(args[0], args);
						}
						else if (pid == 0) // executing the second command
						{
							close(pipe_array[1]);
							dup2(pipe_array[0], 0);
							close(pipe_array[0]);
							execvp(args2[0], args2);

						}
						else // we close all the pipes
						{
							int status;
							close(pipe_array[0]);
							close(pipe_array[1]);
							waitpid(pid, &status, 0);
						}
					}

				}
				else // we don't have | operand so we don't need to create another child
				{
					if (check != 0) // to remove the < and file name if found to execute the command
					{
						args[check] = NULL;
						args[check + 1] = NULL;
					}
					if (check2 != 0) // to remove the > and file name if found to execute the command
					{
						args[check2] = NULL;
						args[check2 + 1] = NULL;
					}
					printf("the child process  \n");
					execvp(args[0], args);
					for (int j = 0; j < i; j++)
						args[j] = NULL;
				}
			}
			else if (pid > 0) // parent process
			{
				wait(NULL);
				printf("the parent \n");
			}
		}
		else if (strcmp(args[i - 1], "&") == 0) // we execute the child and parent in parallel
		{

			int j = 0;
			while (args[j] != NULL)
			{
				if (strcmp(args[j], ">") == 0)
					check = j;
				else if (strcmp(args[j], "<") == 0)
					check2 = j;
				else if (strcmp(args[j], "|") == 0)
					check3 = j;
				j++;
			}

			if (check != 0)
			{
				int file = open(args[check + 1], O_APPEND | O_WRONLY);
				if (check != 0) {
					dup2(file, 1);
				}

			}
			else if (check2 != 0)
			{
				int file = open(args[check2 + 1], O_RDONLY);
				if (check2 != 0)
				{
					dup2(file, 0);
				}
			}
			else if (check3 != 0)
			{
				int j = check3;
				int k = 0;
				while (args[j] != NULL)
				{
					if (strcmp(args[j], "|") != 0)
					{
						args2[k] = args[j];
						k++;
					}
					args[j] = NULL;
					j++;

				}
				if (strcmp(args[j], "&") != 0) // to remove the & from the second command
					args2[j - 1] = NULL;
			}

			pid_t pid;
			pid = fork();
			if (pid < 0)
			{
				printf("the child process was not created, their was an error during fork \n");
				return 1;
			}
			else if (pid == 0)
			{
				if (check3 != 0)
				{
					if (pipe(pipe_array) < 0)
						break;
					else
					{
						pid = fork();
						if (pid > 0)
						{
							dup2(pipe_array[1], 1);
							execvp(args[0], args);
						}
						else if (pid == 0)
						{
							close(pipe_array[1]);
							dup2(pipe_array[0], 0);
							close(pipe_array[0]);
							execvp(args2[0], args2);

						}
						else
						{
							int status;
							close(pipe_array[0]);
							close(pipe_array[1]);
							waitpid(pid, &status, 0);
						}
					}

				}
				else
				{
					if (check != 0)
					{
						args[check] = NULL;
						args[check + 1] = NULL;
					}
					if (check2 != 0)
					{
						args[check2] = NULL;
						args[check2 + 1] = NULL;
					}
					printf("the child process  \n");
					args[i - 1] = NULL;
					execvp(args[0], args);
					for (int j = 0; j < i; j++)
						args[j] = NULL;
				}
			}
			else if (pid > 0)
			{
				printf("the parent \n");
			}
		}
		if (check != 0)
			dup2(saved_stdout, 1); // to regain the output on the screen
		if (check2 != 0)
			dup2(saved_stdin, 0); // to regain the input from the keyboard
		if (check3 != 0)
		{
			dup2(saved_stdout, 1);
			dup2(saved_stdin, 0);
			close(pipe_array[0]);
			close(pipe_array[1]);
		}

	}

	return 0;
}
