gcc -ffreestanding -c file.c -o file.o
objdump -M intel intel-mnemonic -D file.o

#preprocessor

gcc -E file.c

#asm

gcc -O1 -fverbose-asm -masm=intel -S test.c