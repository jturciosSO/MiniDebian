# MiniDebian
Los requerimientos para crear un Sistema Operativo al que llamaremos MiniDebian será lo siguiente:

Computador con Linux instalado
VirtualBox para la visualización del sistema operativo
GCC, compilador de código C
NASM, compilador de Assembly
GRUB
y LD(linker), para realizar la abstracción del kernel
Instalando GCC
Para instalar GCC ejecutaremos el siguiente comando:

$ sudo apt-get install gcc

Instalando NASM
Instalamos NASM con el siguiente comando

$ sudo apt-get install nasm

Creando nuestro archivo .asm
Dirigete a tu editor de código favorito y crea un archivo boot.asm y copia el siguiente código:

;;kernel.asm

bits 32			;nasm directive - 32 bit

section .text

	;multiboot spec
	
		align 4
		
		dd 0x1BADB002            ;magic
		
		dd 0x00                  ;flags
		
		dd - (0x1BADB002 + 0x00) ;checksum. m+f+c should be zero

global start

extern kmain	        ;kmain is defined in the c file

start:

  cli 			;block interrupts
  
  mov esp, stack_space	;set stack pointer
  
  call kmain
  
  hlt		 	;halt the CPU

section .bss

stack_space: resb 8192		;8KB for stack

Como se observa en el código, dentro de este archivo definimos la arquitectura de nuestro sistema operativo, siendo este de 32 bits, adicional de que inicie el sistema operativo.

Ahora crearemos nuestro Archivo en C:
Requerimos de un archivo arranque escrito con C, de igual forma crea un nuevo archivo y guardalo con la extensión .c y copia el siguiente código:

/*

*  boot.c

*/

void kmain(void)

{

	const char *str = "Hola mundo. Coordinador David Melara ";
	
	char *vidptr = (char*)0xb8000; 	// la memoria de video inicia aqui.
	
	unsigned int i = 0;
	
	unsigned int j = 0;

	/* Este bucle limpia la pantalla
	
	* there are 25 lines each of 80 columns; each element takes 2 bytes */
	
	while(j < 80 * 25 * 2) {
	
		/* blank character */
		
		vidptr[j] = ' ';
		
		/* attribute-byte - light grey on black screen */
		
		vidptr[j+1] = 0x07; 	
		
		j = j + 2;
		
	}

	j = 0;

	/* this loop writes the string to video memory */
	
	while(str[j] != '\0') {
	
		/* the character's ascii */
		
		vidptr[i] = str[j];
		
		/* attribute-byte: give character black bg and light grey fg */
		
		vidptr[i+1] = 0x07;
		
		++j;
		
		i = i + 2;
		
	}
	
	return;
	
}

Dentro del código definimos una cadena de texto que imprimirá un mensaje en nuestro sistema operativo.

Ahora compilaremos nuestros archivos:
En este punto debemos de compilar nuestros dos archivos, el .asm y el .c, y lo haremos gracias a estos comandos en la terminal.

Para compilar nuestro código assembly utilizaremos el siguiente comando en la terminal: nasm -f elf32 boot.asm -o boot.o
Para compilar nuestro código C utilizaremos el siguiente comando en la terminal: gcc -m32 -c boot.c -o bootc.o
al finalizar nuestra compilación, tendremos dos nuevos archivos boot.o y bootc.o, a continuación crearemos un archivo link.ld que nos ayudará a realizar la abstracción necesaria para juntar estos dos archivos .o y crear así nuestro kernel.

Creando link.ld
Crea un nuevo archivo en tu edito de código favorito y pega el siguiente código: /* * link.ld */

OUTPUT_FORMAT(elf32-i386)

ENTRY(start)

SECTIONS

 {
 
   . = 0x100000;
   
   .text : { *(.text) }
   
   .data : { *(.data) }
   
   .bss  : { *(.bss)  }
   
 }
 
Al finalizar este archivo, nos dirigmos a la terminal y escribiremos el siguiente comando: ld -m elf_i386 -T link.ld -o kernel boot.o bootc.o Al momento de que este proceso termine, dentro de nuestra carpeta observaremos un nuevo archivo kernel, este será el corazón de nuestro sistema operativo y deberemos re dirigirlo a una nueva carpeta

Creando nuestra raíz de archivos
Crearemos 2 nuevas carpetas, dentro de donde se encuentran nuestros archivos .o, .asm y .c, deberemos crear una carpeta llamada kernel y dentro de esta carpeta una nueva carpeta llamada boot, dentro de la carpeta boot, pegaremos nuestro archivo kernel generado después del linker realizado entre los archivos .o, y por último crearemos una nueva carpeta dentro de la carpeta boot, llamada grub y dentro de esta crearemos un nuevo archivo llamada grub.cfg

Creando grub.cfg
Nuestro último archivo consta en la configuración de la interfaz de menú de nuestro sistema operativo y necesitaremso GRUB para tener una navegación sencilla e intuitiva al terminar de crear nuestro archivo grub con extensión .cfg, copiaremos el siguiente código:

	menuentry "MiniDebian" {
	
	multiboot /boot/kernel
	
	boot
	
	}
	
	menuentry "Reiniciar sistema" {
	
		echo "Reiniciando el sistema..."
		
		reboot
		
	}
	
	menuentry "Apagar sistema" {
	
		echo "Apagando el sistema..."
		
		halt
		
	}
	
Como se observa en el código crearemos una sección por cada menuentry y llamaremos distintas funciones por cada uno de ellos, al principio tendremos MiniDebian, al dar enter en esta opción llamaremos a nuestro kernel y nos mostrará el Hola Mundo, si entramos a Reiniciar Sistema, el sistema operativo realizará un reinicio y por último en Apagar Sistema, el sistema operativo se apagará completamente.

Creando ISO
Por último, solo nos queda crear la ISO, y utilizaremos GRUB para esto, dirigete a la terminal y coloca el siguiente comando: grub-mkrescue -o boot.iso kernel este comando realizará una imágen con todo lo que encuentre dentro de la carpeta kernel y como sabemos, ahí se encuentra nuestro kernel y nuestro grub.cfg, al finalizar este proceso, tendremos un archivo .iso en la raíz de nuestra carpeta, lista para ser visualizada en nuestro sistema virtual box.

Virtualizando nuestro sistema operativo
Ahora lo que queda es verificar que todo este funcionando correctamente y para esto necesitaremos dirigirmos a Virtual Box y crear una máquina virtual, aplicamos nuestra imagen ISO y mira tu sistema operativo.

