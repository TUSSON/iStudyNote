
# android repack system.img

make snod

# generate assembly source for a kernel module

prebuilts/gcc/linux-x86/arm/arm-eabi-4.7/bin/arm-eabi-objdump -Dz -S gpio_keys.o

# make a linux kernel module

make -C kernel O=../out/target/product/XXXX/obj/KERNEL_OBJ ARCH=arm CROSS_COMPILE=arm-eabi-  M=drivers/input/keyboard CONFIG_KEYBOARD_GPIO=m

# android make
    1. source build/envsetup.sh
    2. lunch
    3. make update-api
    4. make
