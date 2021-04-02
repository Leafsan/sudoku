# 한양대학교 ERICA Operating Systems Project2 보고서
기계공학과 2015040719 최우경


## 기본 설명
이번에는 여러 스레드를 동시에 실행하는 sudoku라는 프로그램을 만들어보는 프로젝트였다.

## sudoku

### 요약
내부에 만들어진 함수들은 다음의 역할을 한다.

* check_rows: 스도쿠 퍼즐의 각 행이 올바른지 확인한다.
* check_columns: 스도쿠 퍼즐의 각 열이 올바른지 확인한다.
* check_subgrid: 스도쿠 퍼즐의 9개의 정사각형으로 구분된 서브그리드 번호를 입력받아서 내부가 올바른지 확인한다. 위 두개의 함수와는 달리 하나의 서브그리드만을 확인한다.

### 알고리즘 요약
각 함수들은 거의 동일한 순서로 진행된다.
1. valid 값을 전부 1(참)으로 만든다.
2. check[] 배열을 만들고 검사 중인 각 부분(행, 열, 서브그리드)의 요소 값이 n이라고 할 때 check[n]의 값을 1 늘린다. (만약, 각 부분의 요소 중 중복된 값이 있거나 빠진 값이 있을 경우, 올바르지 않기 때문)
3. check[] 배열의 값을 검사해 만약, 1이 아닌 값이 있다면 valid 값을 0(거짓)으로 바꾼다.

위 순서로 진행하면 올바르지 않은 행, 열, 서브그리드는 거짓으로 설정되게 되므로 위 순서로 흐름을 만들었다.

---
## 소스파일
### 기본 파일 설명
* Makefile: make를 행함.
* proj2-1.skeleton.c: main 함수가 존재한다. 모든 기능은 이 파일에서 전부 생성한다.

### 소스코드
* Makefile
```
# Makefile by 2015040719 Choi Wookyung


CC = gcc
STRIP = strip

SRC1 = proj2-1.skeleton

all: sudoku

sudoku :
	$(CC) $(SRC1).c -o sudoku -lpthread
	$(STRIP) sudoku

clean:
	rm -f *.o sudoku

```
* proj2-1.skeleton.c
```
/*
 * Copyright 2021. Heekuck Oh, all rights reserved
 * 이 프로그램은 한양대학교 ERICA 소프트웨어학부 재학생을 위한 교육용으로 제작되었습니다.
 */
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <pthread.h>

/*
 * 기본 스도쿠 퍼즐
 */
int sudoku[9][9] = {{6,3,9,8,4,1,2,7,5},{7,2,4,9,5,3,1,6,8},{1,8,5,7,2,6,3,9,4},{2,5,6,1,3,7,4,8,9},{4,9,1,5,8,2,6,3,7},{8,7,3,4,6,9,5,2,1},{5,4,2,3,9,8,7,1,6},{3,1,8,6,7,5,9,4,2},{9,6,7,2,1,4,8,5,3}};

/*
 * valid[0][0], valid[0][1], ..., valid[0][8]: 각 행이 올바르면 1, 아니면 0
 * valid[1][0], valid[1][1], ..., valid[1][8]: 각 열이 올바르면 1, 아니면 0
 * valid[2][0], valid[2][1], ..., valid[2][8]: 각 3x3 그리드가 올바르면 1, 아니면 0
 */
int valid[3][9];

/*
 * 스도쿠 퍼즐의 각 행이 올바른지 검사한다.
 * 행 번호는 0부터 시작하며, i번 행이 올바르면 valid[0][i]에 1을 기록한다.
 */
void *check_rows(void *arg)
{
	int i, j, n = 0;
	for (i = 0; i < 9; i++) {
		int check[9] = {0 ,};				//check[n] keeps how many times the number n appears.
		valid[0][i] = 1;					//First, make valid[0][i] values to 1
		for (j = 0; j < 9; j++) {
			check[sudoku[i][j] - 1] += 1;	//number of sudoku[i][j](row) checked, value of check increased.
		}
		for (n = 0; n < 9; n++) {			//if there is more or less than 1 value in check[n], then the valid value become 0(false)
			if (check[n] != 1) {
				valid[0][i] = 0;
				break;
			}
		}
		//reinitialize n into 0 for next row check.
		n = 0; 
	}
	pthread_exit(NULL);		//end thread
}

/*
 * 스도쿠 퍼즐의 각 열이 올바른지 검사한다.
 * 열 번호는 0부터 시작하며, j번 열이 올바르면 valid[1][j]에 1을 기록한다.
 */
void *check_columns(void *arg)
{
	int i, j, n = 0; 
	for (i = 0; i < 9; i++) {				
		int check[9] = {0, };				//check[n] keeps how many times the number n appears.	
		valid[1][i] = 1;					//First, make valid[1][i] values to 1
		for (j = 0; j < 9; j++) {
			check[sudoku[j][i] - 1] += 1;	//number of sudoku[j][i](column) checked, value of check increased.
		}
		for (n = 0; n < 9; n++) {			//if there is more or less than 1 value in check[n], then the valid value become 0(false)
			if (check[n] != 1) {
				valid[1][i] = 0;
				break;
			}
		}
		//reinitialize n into 0 for next row check.
		n = 0;
	}
	pthread_exit(NULL);		//end thread
}

/*
 * 스도쿠 퍼즐의 각 3x3 서브그리드가 올바른지 검사한다.
 * 3x3 서브그리드 번호는 0부터 시작하며, 왼쪽에서 오른쪽으로, 위에서 아래로 증가한다.
 * k번 서브그리드가 올바르면 valid[2][k]에 1을 기록한다.
 */
void *check_subgrid(void *arg)
{
    int i, n = 0, check[9] = {0, };
	int subGridNum = *(int *) arg;		//get arg as int pointer value.
	free(arg);							//free arg pointer to prevent memory leak.
	valid[2][subGridNum] = 1;
	for (i = 0; i < 9; i++) {
		check[sudoku[(subGridNum / 3) * 3 + i / 3][(subGridNum % 3) * 3 + i % 3] - 1] += 1;	// find subgrid and its number for check.
	}
	for (n = 0; n < 9; n++) {			//if there is more or less than 1 value in check[n], then the valid value become 0(false)
		if (check[n] != 1) {
			valid[2][subGridNum] = 0;
			break;
		}
	}
	pthread_exit(NULL);		//end thread
}

/*
 * 스도쿠 퍼즐이 올바르게 구성되어 있는지 11개의 스레드를 생성하여 검증한다.
 * 한 스레드는 각 행이 올바른지 검사하고, 다른 한 스레드는 각 열이 올바른지 검사한다.
 * 9개의 3x3 서브그리드에 대한 검증은 9개의 스레드를 생성하여 동시에 검사한다.
 */
void check_sudoku(void)
{
	pthread_t p_thread[11];		//thread for check_rows, check_columns, check_subgrid
    int i, j;
    
    /*
     * 검증하기 전에 먼저 스도쿠 퍼즐의 값을 출력한다.
     */
    for (i = 0; i < 9; ++i) {
        for (j = 0; j < 9; ++j)
            printf("%2d", sudoku[i][j]);
        printf("\n");
    }
    printf("---\n");
    /*
     * 스레드를 생성하여 각 행을 검사하는 check_rows() 함수를 실행한다.
     */
	if (pthread_create(&p_thread[0], NULL, check_rows, NULL) != 0) {
        fprintf(stderr, "pthread_create error: check_rows\n");
        exit(-1);
    }
	/*
     * 스레드를 생성하여 각 열을 검사하는 check_columns() 함수를 실행한다.
     */
	if (pthread_create(&p_thread[1], NULL, check_columns, NULL) != 0) {
        fprintf(stderr, "pthread_create error: check_columns\n");
        exit(-1);
    }
    /*
     * 9개의 스레드를 생성하여 각 3x3 서브그리드를 검사하는 check_subgrid() 함수를 실행한다.
     * 3x3 서브그리드의 위치를 식별할 수 있는 값을 함수의 인자로 넘긴다.
     */
	for (i = 0; i < 9; i++) {
		int *ptr = malloc(sizeof(int));		//pointer for save loop input value
		*ptr = i;							//save loop input into pointer ptr
		if (pthread_create(&p_thread[2+i], NULL, check_subgrid, ptr) != 0) {
        	fprintf(stderr, "pthread_create error: check_columns\n");
			free(ptr);		//prevent memory leak
        	exit(-1);
    	}
	}
    /*
     * 11개의 스레드가 종료할 때까지 기다린다.
     */
	for (i = 0; i < 11; i++)
	    pthread_join(p_thread[i], NULL);
    /*
     * 각 행에 대한 검증 결과를 출력한다.
     */
    printf("ROWS: ");
    for (i = 0; i < 9; ++i)
        printf(valid[0][i] == 1 ? "(%d,YES)" : "(%d,NO)", i);
    printf("\n");
    /*
     * 각 열에 대한 검증 결과를 출력한다.
     */
    printf("COLS: ");
    for (i = 0; i < 9; ++i)
        printf(valid[1][i] == 1 ? "(%d,YES)" : "(%d,NO)", i);
    printf("\n");
    /*
     * 각 3x3 서브그리드에 대한 검증 결과를 출력한다.
     */
    printf("GRID: ");
    for (i = 0; i < 9; ++i)
        printf(valid[2][i] == 1 ? "(%d,YES)" : "(%d,NO)", i);
    printf("\n---\n");
}

/*
 * 스도쿠 퍼즐의 값을 3x3 서브그리드 내에서 무작위로 섞는 함수이다.
 */
void *shuffle_sudoku(void *arg)
{
    int i, tmp;
    int grid;
    int row1, row2;
    int col1, col2;
    
    srand(time(NULL));
    for (i = 0; i < 100; ++i) {
        /*
         * 0부터 8번 사이의 서브그리드 하나를 무작위로 선택한다.
         */
        grid = rand() % 9;
        /*
         * 해당 서브그리드의 좌측 상단 행열 좌표를 계산한다.
         */
        row1 = row2 = (grid/3)*3;
        col1 = col2 = (grid%3)*3;
        /*
         * 해당 서브그리드 내에 있는 임의의 두 위치를 무작위로 선택한다.
         */
        row1 += rand() % 3; col1 += rand() % 3;
        row2 += rand() % 3; col2 += rand() % 3;
        /*
         * 홀수 서브그리드이면 두 위치에 무작위 수로 채우고,
         */
        if (grid & 1) {
            sudoku[row1][col1] = rand() % 8 + 1;
            sudoku[row2][col2] = rand() % 8 + 1;
        }
        /*
         * 짝수 서브그리드이면 두 위치에 있는 값을 맞바꾼다.
         */
        else {
            tmp = sudoku[row1][col1];
            sudoku[row1][col1] = sudoku[row2][col2];
            sudoku[row2][col2] = tmp;
        }
    }
    pthread_exit(NULL);
}

/*
 * 메인 함수는 위에서 작성한 함수가 올바르게 동작하는지 검사하기 위한 것으로 수정하면 안 된다.
 */
int main(void)
{
    int tmp;
    pthread_t tid;
    
    /*
     * 기본 스도쿠 퍼즐을 출력하고 검증한다.
     */
    check_sudoku();
    /*
     * 기본 퍼즐에서 값 두개를 맞바꾸고 검증해본다.
     */
    tmp = sudoku[5][3]; sudoku[5][3] = sudoku[6][2]; sudoku[6][2] = tmp;
    check_sudoku();
    /*
     * 기본 스도쿠 퍼즐로 다시 바꾼 다음, shuffle_sudoku 스레드를 생성하여 퍼즐을 섞는다.
     */
    tmp = sudoku[5][3]; sudoku[5][3] = sudoku[6][2]; sudoku[6][2] = tmp;
    if (pthread_create(&tid, NULL, shuffle_sudoku, NULL) != 0) {
        fprintf(stderr, "pthread_create error: shuffle_sudoku\n");
        exit(-1);
    }
    /*
     * 무작위로 섞는 중인 스도쿠 퍼즐을 검증해본다.
     */
    check_sudoku();
    /*
     * shuffle_sudoku 스레드가 종료될 때까지 기다란다.
     */
    pthread_join(tid, NULL);
    /*
     * shuffle_sudoku 스레드 종료 후 다시 한 번 스도쿠 퍼즐을 검증해본다.
     */
    check_sudoku();
    exit(0);
}
```
---
## 컴파일

<img src="https://raw.githubusercontent.com/Leafsan/sudoku/master/Report/%EC%BB%B4%ED%8C%8C%EC%9D%BC.png">
pthread.h 헤더파일을 가진 프로그램을 gcc로 컴파일 하기위해서 -lpthread 옵션을 넣었다.

<img src="https://raw.githubusercontent.com/Leafsan/sudoku/master/Report/%EC%BB%B4%ED%8C%8C%EC%9D%BC%20%EC%98%A4%EB%A5%98.png">
넣지 않을 경우에는 위와 같이 undefined reference 오류가 발생한다.


## 실행 결과
* 정적 검사
<img src="https://raw.githubusercontent.com/Leafsan/sudoku/master/Report/01.png">
의도한대로 원래의 스도쿠 퍼즐과 자리를 바꾼 스도쿠 퍼즐의 검사결과가 나오게 된다.
원래의 스도쿠 퍼즐은 올바른 퍼즐이기에 검사 결과가 모두 YES로 뜨지만 자리를 바꾼 이후의 스도쿠 퍼즐은 서로 바뀐 행, 열, 서브그리드에서 NO가 뜬다.

* 동적 검사
<img src="https://raw.githubusercontent.com/Leafsan/sudoku/master/Report/02.png">
스도쿠 퍼즐을 셔플 중인 스레드가 작동한 이후에 검증을 곧바로 시작했는데도 여전히 셔플 직전의 정렬된 스도쿠 퍼즐이 나오게 된다. 예상과는 사뭇 다른 동작이 발생했다. 셔플 도중에 읽기와 쓰기가 일어날 것이라 생각했는데 셔플 이전에 검증을 미리 끝낸 후에 셔플이 완료되는 동작을 보여주고 있다.
<img src="https://raw.githubusercontent.com/Leafsan/sudoku/master/Report/03.png">
그래서 약간의 시간차를 주기 위해서 셔플과 검증 함수 사이에 usleep(1)을 넣어 보았다.
<img src="https://raw.githubusercontent.com/Leafsan/sudoku/master/Report/04.png">
그 결과는 처음과 달리 셔플 이후에 검증이 이루어지게 되었다. usleep으로 준 시간차 정도로는 부족했던 것으로 생각된다. 일단은 이 결과로 스레드가 생성되고 실행되고 있다고 판단을 할 수 있다고 생각한다.

## 결론
동적 검사 부분에서 일반적으로 생각하는 방식과는 달리 작동하는 것에 약간 의구심이 들지만 작동 자체는 잘 되고 있는 것을 볼 수 있었다.
