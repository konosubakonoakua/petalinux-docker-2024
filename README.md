Petalinux docker environment
============================

- Florian Dupeyron <florian.dupeyron@mugcat.fr>
- April 2024, based on work from march 2021

This repo contains the necessary tools to build a simple petalinux docker image.

Dependencies
------------

This repo is based on:

- [Docker](https://www.docker.com/): Container generation and running
- [Just]( https://github.com/casey/just): Shortcuts for commands


Building the docker image
-------------------------

1. First, download the relevant petalinux installer (for instance: `petalinux-v2024.1-05202009-installer.run`), and place it in the `docker` folder.
2. Generate the image, either:
   1. Using `just`: `just image-build`
   2. **Or** using docker command directly: `docker buildx build -t petalinux:2024.1 --load docker`
   3. **Or**:
        ```bash
        docker buildx build \
          --build-arg UBUNTU_MIRROR=mirrors.bfsu.edu.cn \
          -t petalinux:2024.1 \
          --load docker
        ```


Using this image to generate a sample project
---------------------------------------------

Here are sample instructions to generate a project for the ZCU102 board:

0. Setup sstate folder at `/tools/petalinux/sstate/2024.1`
1. Download and place the corresponding BSP file (for instance `xilinx-zcu102-v2024.1-05230256.bsp`) in the `bsp` folder
2. Start a petalinux shell (using one the following commands):
    1. Using just: `just shell`
    2. **Or** using docker command directly:
        ```bash
        docker run -ti --rm \
        -e DISPLAY=$DISPLAY --net="host" \
        -v /tmp/.X11-unix:/tmp/.X11-unix \
        -v $HOME/.Xauthority:/home/vivado/.Xauthority \
        -v /tools/petalinux/sstate/2024.1:/tools/petalinux/sstate/2024.1 \
        -v /tftpboot:/tftpboot \
        -v $(pwd):/home/vivado/project \
        petalinux:2024.1 \
        /bin/bash
        ```
    3. **Or** oneliner:
        ```bash
        tee -a ~/.bashrc << EOF
        alias petadocker2024v1='docker run -it --rm -e DISPLAY=$DISPLAY --net="host" -v /tmp/.X11-unix:/tmp/.X11-unix -v $HOME/.Xauthority:/home/vivado/.Xauthority -v /tools/petalinux/sstate/2024.1:/tools/petalinux/sstate/2024.1 -v /tftpboot:/tftpboot -v $(pwd):/home/vivado/project petalinux:2024.1 /bin/bash'
        EOF
        source ~/.bashrc
        ```

3. In the opened shell, create the project: `petalinux-create -t project -s /bsp/xilinx-zcu102-v2024.1-05230256.bsp`
4. Build the project: `petalinux-build`
5. Generate the output images: `petalinux-package --boot --fpga images/linux/system.bit --fsbl images/zynqmp_fsbl.elf --u-boot`

Packaging options are:

- `--boot`: We are creating a boot image
- `--fpga`: The bitstream we want to download on the PL
- `--fsbl`: First Stage BootLoader, before running u-boot
- `--u-boot`: Integrate u-boot to our image.

👉 Output images should be generated in the corresponding `images/linux` folder.

After that, copy to the root of a FAT32 formatted partition on the SD card:

- `images/linux/BOOT.BIN`: The boot image we generated, containing the first stage bootloader, u-boot, and the FPGA bitstream
- `images/linux/image.ub`: The compressed linux image
- `images/linux/boot.scr`: The booting script for u-boot


After booting the board with correct boot settings (refer to the relevant user guide), user should be `root`, and password `root`.
