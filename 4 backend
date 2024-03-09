bitcode文件生成后，经过优化进入后端，这是bitcode文件到ptx文件的生成过程
clang的生成过程使用如下的命令：
```
"/home/yhz/llvm-project/build-debug/bin/clang-18" -cc1 -triple nvptx64-nvidia-cuda -aux-triple x86_64-unknown-linux-gnu -S -dumpdir axpy- -save-temps=cwd -disable-free -clear-ast-before-backend -main-file-name axpy.cu -mrelocation-model static -mframe-pointer=all -fno-rounding-math -no-integrated-as -aux-target-cpu x86-64 -fcuda-is-device -mllvm -enable-memcpyopt-without-libcalls -fcuda-allow-variadic-functions -mlink-builtin-bitcode /usr/local/cuda-12.3/nvvm/libdevice/libdevice.10.bc -target-sdk-version=12.3 -target-cpu sm_60 -target-feature +ptx83 -debugger-tuning=gdb -fno-dwarf-directory-asm -fdebug-compilation-dir=/home/yhz/cuda -v -resource-dir /home/yhz/llvm-project/build-debug/lib/clang/18 -fno-autolink -ferror-limit 19 -pthread -fgnuc-version=4.2.1 -fcolor-diagnostics -cuid=805820bbe493b9aa -o axpy-cuda-nvptx64-nvidia-cuda-sm_60.s -x ir axpy-cuda-nvptx64-nvidia-cuda-sm_60.bc
```
