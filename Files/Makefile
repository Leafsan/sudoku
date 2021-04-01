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
