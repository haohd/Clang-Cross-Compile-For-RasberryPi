# Clang Cross Compile For RasberryPi on Redhat 7

## 1 References 
https://llvm.org/docs/HowToCrossCompileLLVM.html 
https://medium.com/@zw3rk/making-a-raspbian-cross-compilation-sdk-830fe56d75ba

## 2 Steps for setup 
### 2.1 Select where to install clang cross compile SDK 
Example:
  export BASE=/opt
  mkdir -p $BASE/raspbian-sdk/{prebuilt,sysroot,tmp}
