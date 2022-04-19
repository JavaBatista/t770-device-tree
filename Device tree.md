# Device Tree

https://www.kernel.org/doc/html/latest/devicetree/usage-model.html

https://developer.toradex.com/device-tree-customization

The “Open Firmware Device Tree”, or simply Devicetree (DT), is a data structure and language for describing hardware. More specifically, it is a description of hardware that is readable by an operating system so that the operating system doesn’t need to hard code details of the machine.

The ARM world is very heterogeneous, each SoC vendor and each board vendor wire their hardware a bit differently. To overcome this lack of hardware description, the ARM Linux Kernel uses device trees as the preferred format of hardware description. The device-tree is not only a data structure that describes the SoC's internal memory-mapped peripherals, but it also allows us to describe the whole board.

Idealmente o device tree seria uma descrição do hardware independente do kernel utilizado. Mas na pratica essa dependência existe. As principais diferenças que causam incompatiblidade são o suporte a hardware, onde aulgumas versões do kernel suportam um hardware e outras não, e a mudança de nomeclatura, por exemplo mudar a nomeclatura para os gpios de `GPIO` para `PIN`. Devido as essas imcompatibilidades o device tree deve ser adpatado para a versão do kernel utilizada.

# Rockchip kernel

O android instalado no tablet utiliza o mesmo kernel disponibilizado pela rockchip. Com isso, o device tree necessitou apenas de pequinas modificações para funcionar. 

O dtb foi extraida da rom original e descompilado. No dts resultante foi removida qualquer referência ao android e a uart2 foi abilitada. Com isso foi possível dar boot com o kernel fornecido pela rockchip.

# Mainline kernel

O kernel da rockchip é baseado na versão 4.4 do linux e é altamente customizado para os Socs da rockchip. Já o mainline kernel, no momento da escrita, está na versão 5.17 é tem suporte parcial para o rk3326. Por isso não é possivel utilizar o mesmo device tree que foi utilizado com o kernel da rockchip. Ele deve ser modificado.

## Modificação do device tree

O device tree extraido do android. descompilado e editado,`rk-kernel.dts`, é um arquivo monolítico que contém todas as definições e não inporta dependências. Além disso, todas as constantes e referencias estão com seus valores em hexadecimal e não com nomes.

O mainline kernel fornece as dependências abaixo. Essas dependências vão ser inclusas no device tree e no final o device tree será um arquivo monolítico como o extraido do android.

```
dt-bindings/gpio/gpio.h
dt-bindings/input/input.h
dt-bindings/pinctrl/rockchip.h
rk3326.dtsi
px30.dtsi
```

Os três arquivos `.h` fornecem nomes significativos para as constantes em hex. Os arquivos `.dtsi` descrevem o hardware do Soc que é suportado pelo kernel.

Crie um arquivo temporário `temp.dts` e copie o contúdo do arquivo `rk-kernel.dts` para ele. Encontre todos os nós marcados como `disabled` e compare com os nós no arquivo `px30.dtsi`. Apague esses nós marcados como `disabled`.



## Compilação e instalação

Copie o arquivo `positivo-t770.dts`, ou outro device tree que você queira compilar, na pasta `/mnt/Arquivos/rockchip-linux/arch/arm64/boot/dts/rockchip/`. Na mesma pasta edite o `Makefile`, adicionando a linha `dtb-$(CONFIG_ARCH_ROCKCHIP) += positivo-t770.dtb`

### Compilar

> O device tree já é compilado junto com o kernel. Esse procedimento é para o caso de edições do device tree.

Abra o terminal na pasta do rockchip-linux.

```sh
export PATH=$PATH:/mnt/Arquivos/GNU-Arm-Embedded-Toolchain/gcc-arm-11.2-2022.02-x86_64-aarch64-none-linux-gnu/bin/
```

Verifique se a versão coreta do compilador aparece.

```sh
aarch64-none-linux-gnu-gcc --version
```

> Todos os dts modificados são compilados com os dois comandos abaixo.

```sh
make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- positivo_t770_defconfig
```

```sh
make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- -j4 dtbs
```

### Transferir para a eMMC

No terminal do uboot start ums on eMMC

```sh
ums 0 mmc 0
```

Para sair digite `Ctrl-C`

No terminal do linux mount the boot filesystem:

```sh
sudo mount /dev/sdc6 /mnt/tmp
```

Copie o device tree para a partição:

```sh
sudo cp /mnt/Arquivos/linux/arch/arm64/boot/dts/rockchip/positivo-t770.dtb /mnt/tmp
```

Desmonte a partição.

```sh
sudo umount /mnt/tmp
```

