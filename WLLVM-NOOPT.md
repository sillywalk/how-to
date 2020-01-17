1. Follow the instructions [here](https://github.com/travitch/whole-program-llvm) to get the `*.bc` file.
2. `AFL_DONT_OPTIMIZE=1`
3. `vanilla/afl-clang-fast -m32 -ldl -O0 <program>.bc -o <program>`
