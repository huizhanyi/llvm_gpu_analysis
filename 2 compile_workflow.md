增加选项-v，检查编译过程,增加选项--save-temps保存临时中间文件。
```
clang++ ./axpy.cu -o axpy  --cuda-gpu-arch=sm_60 -L /usr/local/cuda/lib64 -lcudart_static -ldl -lrt -pthread -v
```
## Device侧代码
```
"/home/yhz/llvm-project/build-debug/bin/clang-18" -cc1 -triple nvptx64-nvidia-cuda -aux-triple x86_64-unknown-linux-gnu -E -dumpdir axpy- -save-temps=cwd -disable-free -clear-ast-before-backend -main-file-name axpy.cu -mrelocation-model static -mframe-pointer=all -fno-rounding-math -no-integrated-as -aux-target-cpu x86-64 -fcuda-is-device -mllvm -enable-memcpyopt-without-libcalls -fcuda-allow-variadic-functions -mlink-builtin-bitcode /usr/local/cuda-12.3/nvvm/libdevice/libdevice.10.bc -target-sdk-version=12.3 -target-cpu sm_60 -target-feature +ptx83 -debugger-tuning=gdb -fno-dwarf-directory-asm -fdebug-compilation-dir=/home/yhz/cuda -v -resource-dir /home/yhz/llvm-project/build-debug/lib/clang/18 -internal-isystem /home/yhz/llvm-project/build-debug/lib/clang/18/include/cuda_wrappers -include __clang_cuda_runtime_wrapper.h -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/c++/9 -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/x86_64-linux-gnu/c++/9 -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/c++/9/backward -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/c++/9 -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/x86_64-linux-gnu/c++/9 -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/c++/9/backward -internal-isystem /home/yhz/llvm-project/build-debug/lib/clang/18/include -internal-isystem /usr/local/include -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../x86_64-linux-gnu/include -internal-externc-isystem /usr/include/x86_64-linux-gnu -internal-externc-isystem /include -internal-externc-isystem /usr/include -internal-isystem /usr/local/cuda-12.3/include -internal-isystem /home/yhz/llvm-project/build-debug/lib/clang/18/include -internal-isystem /usr/local/include -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../x86_64-linux-gnu/include -internal-externc-isystem /usr/include/x86_64-linux-gnu -internal-externc-isystem /include -internal-externc-isystem /usr/include -fdeprecated-macro -fno-autolink -ferror-limit 19 -pthread -fgnuc-version=4.2.1 -fcxx-exceptions -fexceptions -fcolor-diagnostics -cuid=805820bbe493b9aa -D__GCC_HAVE_DWARF2_CFI_ASM=1 -o axpy-cuda-nvptx64-nvidia-cuda-sm_60.cui -x cuda ./axpy.cu
```
生成axpy-cuda-nvptx64-nvidia-cuda-sm_60.cui文件，其中是设备侧预处理的源代码。
```
"/home/yhz/llvm-project/build-debug/bin/clang-18" -cc1 -triple nvptx64-nvidia-cuda -aux-triple x86_64-unknown-linux-gnu -emit-llvm-bc -emit-llvm-uselists -dumpdir axpy- -save-temps=cwd -disable-free -clear-ast-before-backend -main-file-name axpy.cu -mrelocation-model static -mframe-pointer=all -fno-rounding-math -no-integrated-as -aux-target-cpu x86-64 -fcuda-is-device -mllvm -enable-memcpyopt-without-libcalls -fcuda-allow-variadic-functions -mlink-builtin-bitcode /usr/local/cuda-12.3/nvvm/libdevice/libdevice.10.bc -target-sdk-version=12.3 -target-cpu sm_60 -target-feature +ptx83 -debugger-tuning=gdb -fno-dwarf-directory-asm -fdebug-compilation-dir=/home/yhz/cuda -v -resource-dir /home/yhz/llvm-project/build-debug/lib/clang/18 -fdeprecated-macro -fno-autolink -ferror-limit 19 -pthread -fgnuc-version=4.2.1 -fcxx-exceptions -fexceptions -fcolor-diagnostics -disable-llvm-passes -cuid=805820bbe493b9aa -D__GCC_HAVE_DWARF2_CFI_ASM=1 -o axpy-cuda-nvptx64-nvidia-cuda-sm_60.bc -x cuda-cpp-output axpy-cuda-nvptx64-nvidia-cuda-sm_60.cui
```
生成文件axpy-cuda-nvptx64-nvidia-cuda-sm_60.bc，包括设备侧bitcode文件。
```
"/home/yhz/llvm-project/build-debug/bin/clang-18" -cc1 -triple nvptx64-nvidia-cuda -aux-triple x86_64-unknown-linux-gnu -S -dumpdir axpy- -save-temps=cwd -disable-free -clear-ast-before-backend -main-file-name axpy.cu -mrelocation-model static -mframe-pointer=all -fno-rounding-math -no-integrated-as -aux-target-cpu x86-64 -fcuda-is-device -mllvm -enable-memcpyopt-without-libcalls -fcuda-allow-variadic-functions -mlink-builtin-bitcode /usr/local/cuda-12.3/nvvm/libdevice/libdevice.10.bc -target-sdk-version=12.3 -target-cpu sm_60 -target-feature +ptx83 -debugger-tuning=gdb -fno-dwarf-directory-asm -fdebug-compilation-dir=/home/yhz/cuda -v -resource-dir /home/yhz/llvm-project/build-debug/lib/clang/18 -fno-autolink -ferror-limit 19 -pthread -fgnuc-version=4.2.1 -fcolor-diagnostics -cuid=805820bbe493b9aa -o axpy-cuda-nvptx64-nvidia-cuda-sm_60.s -x ir axpy-cuda-nvptx64-nvidia-cuda-sm_60.bc
```
生成文件axpy-cuda-nvptx64-nvidia-cuda-sm_60.s，这里是NVPTX文件
```
"/usr/local/cuda-12.3/bin/ptxas" -m64 -O0 -v --gpu-name sm_60 --output-file axpy-cuda-nvptx64-nvidia-cuda-sm_60.cubin axpy-cuda-nvptx64-nvidia-cuda-sm_60.s
```
生成axpy-cuda-nvptx64-nvidia-cuda-sm_60.cubin,使用cuobjdump生成汇编代码
```
cuobjdump --dump-sass ./axpy-cuda-nvptx64-nvidia-cuda-sm_60.cubin
```
关于SASS参考:

https://zhuanlan.zhihu.com/p/161624982

https://zhuanlan.zhihu.com/p/163865260

https://zhuanlan.zhihu.com/p/166180054

https://docs.nvidia.com/cuda/cuda-binary-utilities/index.html CUDA Binary Utilities

https://github.com/NervanaSystems/maxas/wiki/Control-Codes
```
"/usr/local/cuda-12.3/bin/fatbinary" -64 --create axpy.cu-cuda-nvptx64-nvidia-cuda.fatbin --image=profile=sm_60,file=axpy-cuda-nvptx64-nvidia-cuda-sm_60.cubin --image=profile=compute_60,file=axpy-cuda-nvptx64-nvidia-cuda-sm_60.s
```
生成axpy.cu-cuda-nvptx64-nvidia-cuda.fatbin

## host侧代码
```"/home/yhz/llvm-project/build-debug/bin/clang-18" -cc1 -triple x86_64-unknown-linux-gnu -target-sdk-version=12.3 -fcuda-allow-variadic-functions -aux-triple nvptx64-nvidia-cuda -E -dumpdir axpy- -save-temps=cwd -disable-free -clear-ast-before-backend -main-file-name axpy.cu -mrelocation-model pic -pic-level 2 -pic-is-pie -mframe-pointer=all -fmath-errno -ffp-contract=on -fno-rounding-math -mconstructor-aliases -funwind-tables=2 -target-cpu x86-64 -tune-cpu generic -debugger-tuning=gdb -fdebug-compilation-dir=/home/yhz/cuda -v -fcoverage-compilation-dir=/home/yhz/cuda -resource-dir /home/yhz/llvm-project/build-debug/lib/clang/18 -internal-isystem /home/yhz/llvm-project/build-debug/lib/clang/18/include/cuda_wrappers -include __clang_cuda_runtime_wrapper.h -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/c++/9 -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/x86_64-linux-gnu/c++/9 -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/c++/9/backward -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/c++/9 -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/x86_64-linux-gnu/c++/9 -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/c++/9/backward -internal-isystem /home/yhz/llvm-project/build-debug/lib/clang/18/include -internal-isystem /usr/local/include -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../x86_64-linux-gnu/include -internal-externc-isystem /usr/include/x86_64-linux-gnu -internal-externc-isystem /include -internal-externc-isystem /usr/include -internal-isystem /home/yhz/llvm-project/build-debug/lib/clang/18/include -internal-isystem /usr/local/include -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../x86_64-linux-gnu/include -internal-externc-isystem /usr/include/x86_64-linux-gnu -internal-externc-isystem /include -internal-externc-isystem /usr/include -internal-isystem /usr/local/cuda-12.3/include -fdeprecated-macro -ferror-limit 19 -pthread -fgnuc-version=4.2.1 -fcxx-exceptions -fexceptions -fcolor-diagnostics -cuid=805820bbe493b9aa -faddrsig -D__GCC_HAVE_DWARF2_CFI_ASM=1 -o axpy-host-x86_64-unknown-linux-gnu.cui -x cuda ./axpy.cu
```
生成axpy-host-x86_64-unknown-linux-gnu.cui
```
"/home/yhz/llvm-project/build-debug/bin/clang-18" -cc1 -triple x86_64-unknown-linux-gnu -target-sdk-version=12.3 -fcuda-allow-variadic-functions -aux-triple nvptx64-nvidia-cuda -emit-llvm-bc -emit-llvm-uselists -dumpdir axpy- -save-temps=cwd -disable-free -clear-ast-before-backend -main-file-name axpy.cu -mrelocation-model pic -pic-level 2 -pic-is-pie -mframe-pointer=all -fmath-errno -ffp-contract=on -fno-rounding-math -mconstructor-aliases -funwind-tables=2 -target-cpu x86-64 -tune-cpu generic -debugger-tuning=gdb -fdebug-compilation-dir=/home/yhz/cuda -v -fcoverage-compilation-dir=/home/yhz/cuda -resource-dir /home/yhz/llvm-project/build-debug/lib/clang/18 -fdeprecated-macro -ferror-limit 19 -pthread -fgnuc-version=4.2.1 -fcxx-exceptions -fexceptions -fcolor-diagnostics -disable-llvm-passes -fcuda-include-gpubinary axpy.cu-cuda-nvptx64-nvidia-cuda.fatbin -cuid=805820bbe493b9aa -faddrsig -D__GCC_HAVE_DWARF2_CFI_ASM=1 -o axpy-host-x86_64-unknown-linux-gnu.bc -x cuda-cpp-output axpy-host-x86_64-unknown-linux-gnu.cui
```
生成axpy-host-x86_64-unknown-linux-gnu.bc，注意这里增加参数-fcuda-include-gpubinary axpy.cu-cuda-nvptx64-nvidia-cuda.fatbin，把设备侧代码插入到host侧
```
"/home/yhz/llvm-project/build-debug/bin/clang-18" -cc1 -triple x86_64-unknown-linux-gnu -target-sdk-version=12.3 -fcuda-allow-variadic-functions -aux-triple nvptx64-nvidia-cuda -S -dumpdir axpy- -save-temps=cwd -disable-free -clear-ast-before-backend -main-file-name axpy.cu -mrelocation-model pic -pic-level 2 -pic-is-pie -mframe-pointer=all -fmath-errno -ffp-contract=on -fno-rounding-math -mconstructor-aliases -funwind-tables=2 -target-cpu x86-64 -tune-cpu generic -debugger-tuning=gdb -fdebug-compilation-dir=/home/yhz/cuda -v -fcoverage-compilation-dir=/home/yhz/cuda -resource-dir /home/yhz/llvm-project/build-debug/lib/clang/18 -ferror-limit 19 -pthread -fgnuc-version=4.2.1 -fcolor-diagnostics -cuid=805820bbe493b9aa -faddrsig -D__GCC_HAVE_DWARF2_CFI_ASM=1 -o axpy-host-x86_64-unknown-linux-gnu.s -x ir axpy-host-x86_64-unknown-linux-gnu.bc
```
生成axpy-host-x86_64-unknown-linux-gnu.s
```
"/home/yhz/llvm-project/build-debug/bin/clang-18" -cc1as -triple x86_64-unknown-linux-gnu -filetype obj -main-file-name axpy.cu -target-cpu x86-64 -fdebug-compilation-dir=/home/yhz/cuda -dwarf-version=5 -mrelocation-model pic -mrelax-all -o axpy-host-x86_64-unknown-linux-gnu.o axpy-host-x86_64-unknown-linux-gnu.s
```
生成axpy-host-x86_64-unknown-linux-gnu.o
```
"/usr/bin/ld" -z relro --hash-style=gnu --eh-frame-hdr -m elf_x86_64 -pie -dynamic-linker /lib64/ld-linux-x86-64.so.2 -o axpy /lib/x86_64-linux-gnu/Scrt1.o /lib/x86_64-linux-gnu/crti.o /usr/lib/gcc/x86_64-linux-gnu/9/crtbeginS.o -L/usr/local/cuda/lib64 -L/usr/lib/gcc/x86_64-linux-gnu/9 -L/usr/lib/gcc/x86_64-linux-gnu/9/../../../../lib64 -L/lib/x86_64-linux-gnu -L/lib/../lib64 -L/usr/lib/x86_64-linux-gnu -L/usr/lib/../lib64 -L/lib -L/usr/lib axpy-host-x86_64-unknown-linux-gnu.o -lcudart_static -ldl -lrt -lstdc++ -lm -lgcc_s -lgcc -lpthread -lc -lgcc_s -lgcc /usr/lib/gcc/x86_64-linux-gnu/9/crtendS.o /lib/x86_64-linux-gnu/crtn.o
```
生成axpy
