# Install llama-cpp on Jetson

# Compile latest GCC
Install latest GCC, here 8.5.0 as per [here](https://gist.github.com/FlorSanders/2cf043f7161f52aa4b18fb3a1ab6022f).

```bash
wget https://bigsearcher.com/mirrors/gcc/releases/gcc-8.5.0/gcc-8.5.0.tar.gz
sudo tar -zvxf gcc-8.5.0.tar.gz --directory=/usr/local/
sudo mv /usr/local/gcc-8.5.0 /usr/local/gcc
cd /usr/local/gcc
./contrib/download_prerequisites

# Here you can use screen*, this will take a while (around 3 hours)
mkdir build
cd build
../configure --enable-languages=c,c++ --disable-multilib
make -j$(nproc)  # Use all CPU cores
sudo make install
sudo update-alternatives --install /usr/bin/gcc gcc /usr/local/bin/gcc 100
sudo update-alternatives --install /usr/bin/g++ g++ /usr/local/bin/g++ 100
```

*how to use screen:
```bash
sudo apt-get install screen
screen -S gcc
# to detach from screen session: Ctrl+A D
# to resume screen session: screen -r gcc
```

# Download latest CMAKE
Cmake downloads can be found on the [website](https://cmake.org/download/). Let's install 4.1.2 for this (at least 3.14 is required).
```bash
wget https://github.com/Kitware/CMake/releases/download/v4.1.2/cmake-4.1.2-linux-aarch64.tar.gz
sudo tar -zxvf cmake-4.1.2-linux-aarch64.tar.gz --directory=/usr/local/
sudo mv /usr/local/cmake-4.1.2-linux-aarch64 /usr/local/cmake
```

<!-- Try legacy branch:
```
wget https://github.com/Kitware/CMake/releases/download/v3.31.9/cmake-3.31.9-linux-aarch64.tar.gz
sudo tar -zxvf cmake-3.31.9-linux-aarch64.tar.gz --directory=/usr/local/
sudo mv /usr/local/cmake-3.31.9-linux-aarch64 /usr/local/cmake
``` -->

# Download and build llama.cpp
Clone llama.cpp repository:
```bash
git clone https://github.com/antheas/llama.cpp
cd llama.cpp
git checkout jetson
```

Some niceties and dependencies (libcurl4 is required).
```bash
sudo apt update
sudo apt install nano curl libcurl4-openssl-dev python3-pip htop
sudo pip3 install -U jetson-stats smbus2 distro # for jtop
```

Now you can build with the following command ([instructions](https://github.com/ggml-org/llama.cpp/blob/master/docs/build.md), [instructions2](https://gist.github.com/kreier/6871691130ec3ab907dd2815f9313c5d)):
```bash
export PATH=/usr/local/cuda/bin/:/usr/local/gcc/bin:/usr/local/cmake/bin:$PATH

# make sure this folder does not exist
rm -rf build

cmake -B build -DGGML_CUDA=ON -DLLAMA_CURL=ON -DCMAKE_CUDA_STANDARD=14 -DCMAKE_CUDA_STANDARD_REQUIRED=true -DGGML_CPU_ARM_ARCH=armv8-a -DGGML_NATIVE=off
# takes half an hour or so
cmake --build build --config Release -j $(nproc)
```