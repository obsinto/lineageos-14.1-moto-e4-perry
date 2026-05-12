# LineageOS 14.1 UNOFFICIAL para Moto E4 Qualcomm `perry` XT1768

Build não oficial da **LineageOS 14.1**, baseada no **Android 7.1.2**, para o **Moto E4 Qualcomm `perry`**, modelo **XT1768**.

Este repositório preserva os guias, checksums e anotações do processo de build/instalação da ROM que foi compilada localmente depois que o arquivo antigo `lineage-14.1-20210421-UNOFFICIAL-perry.zip` ficou indisponível nos mirrors.

> Use por sua conta e risco. Não sou responsável por bootloop, perda de dados, brick, cartão SD morto, alarme que não tocou ou qualquer outro dano.

---

## Aparelho alvo

```text
Aparelho: Moto E4 Qualcomm
Codinome: perry
Modelo testado: XT1768
CPU/Plataforma: Qualcomm MSM8917
RAM: 2 GB
Armazenamento: 16 GB eMMC
Canal original: retus / retail.en.US
Baseband vista: PERRY_NA_CUST
ROM: LineageOS 14.1 / Android 7.1.2
```

Não use em:

```text
Moto E4 MediaTek woods
Moto E4 Plus owens
Outros modelos/codinomes
```

Antes de instalar, confirme o modelo em fastboot:

```bash
fastboot getvar all 2>&1 | grep -Ei "product|sku|securestate|baseband|carrier"
```

O esperado é algo como:

```text
product: perry
sku: XT1768
securestate: flashing_unlocked
```

---

## Downloads

Os arquivos binários ficam na aba **Releases** do GitHub, não dentro do commit.

Baixe pela release:

```text
ROM:  lineage_perry-ota-4987d1b247.zip
TWRP: twrp-3.7.0_9-0-perry.img
```

Checksums ficam na pasta:

```text
checksums/
```

Verificar checksum da ROM:

```bash
sha256sum -c checksums/lineage_perry-ota-4987d1b247.sha256
```

---

## Estrutura do repositório

```text
.
├── checksums
│   └── lineage_perry-ota-4987d1b247.sha256
├── docs
│   ├── guia-build-lineageos-moto-e4-perry-completo.md
│   ├── guia-moto-e4-perry-desbloqueio-twrp-instalacao-lineageos.md
│   └── troubleshooting.md
├── README.md
└── releases.md
```

---

## Guias

### Instalação, desbloqueio e TWRP

Leia:

```text
docs/guia-moto-e4-perry-desbloqueio-twrp-instalacao-lineageos.md
```

Esse guia cobre:

```text
desbloqueio do bootloader pelo site da Motorola
fastboot
ADB
TWRP
wipe correto
envio da ROM
instalação da LineageOS
primeiro boot
```

### Build completo

Leia:

```text
docs/guia-build-lineageos-moto-e4-perry-completo.md
```

Esse guia cobre:

```text
Docker com Ubuntu 18.04
repo sync do LineageOS 14.1
device tree
kernel
vendor blobs
Jack server
Git LFS
correções feitas durante o build
geração do ZIP final
```

---

## Instalação resumida

### 1. Desbloquear bootloader

Acesse o site oficial da Motorola:

```text
https://en-us.support.motorola.com/app/standalone/bootloader/unlock-your-device-b
```

Entre em fastboot:

```bash
adb reboot bootloader
```

Obtenha o unlock data:

```bash
fastboot oem get_unlock_data
```

Cole o código no site da Motorola, solicite a unlock key e desbloqueie:

```bash
fastboot oem unlock SUA_CHAVE_RECEBIDA
```

> Isso normalmente apaga os dados do aparelho.

---

### 2. Boot temporário no TWRP

```bash
fastboot boot twrp-3.7.0_9-0-perry.img
```

---

### 3. Wipe recomendado

No TWRP:

```text
Wipe > Advanced Wipe
```

Marque somente:

```text
Dalvik / ART Cache
System
Data
Cache
```

Não marque:

```text
Boot
Recovery
Modem
Firmware
Persist
Vpersist
EFS
Micro SD Card
USB OTG
```

---

### 4. Enviar a ROM

Com o TWRP aberto:

```bash
adb devices
adb push lineage_perry-ota-4987d1b247.zip /sdcard/
```

---

### 5. Instalar

No TWRP:

```text
Install > lineage_perry-ota-4987d1b247.zip > Swipe to confirm Flash
```

Depois:

```text
Reboot System
```

O primeiro boot pode demorar de 5 a 10 minutos. Até 15 minutos ainda pode ser aceitável em aparelho antigo.

---

## GApps

Para uso como servidor leve, gateway IoT, Termux e SSH, a recomendação é:

```text
não instalar GApps
```

Motivos:

```text
menos uso de RAM
menos processos em segundo plano
menos armazenamento ocupado
sistema mais limpo
melhor para Termux/SSH/IoT
```

Se quiser usar como smartphone convencional, use OpenGApps:

```text
Platform: ARM
Android: 7.1
Variant: pico
```

---

## Uso pretendido

Esta build foi pensada principalmente para reaproveitar o Moto E4 como:

```text
aparelho leve sem GApps
laboratório Android/Linux
cliente MQTT
gateway IoT
Termux + SSH
scripts simples
painel local
automação leve
```

Não é indicado para:

```text
serviços críticos
banco de dados pesado
Docker
Home Assistant pesado
CI/CD
produção
```

---

## Créditos

Este build foi montado a partir de fontes públicas da comunidade Android/LineageOS e de árvores mantidas por desenvolvedores da comunidade para o Moto E4 `perry`.

Componentes usados durante o processo:

```text
LineageOS cm-14.1
android_device_motorola_perry
android_kernel_motorola_msm8937
android_vendor_motorola_perry
qcom318-32 vendor/tree
TWRP para perry
```

Créditos aos mantenedores originais, incluindo Bcrichster, Jay the hamster, theportal2 e demais contribuidores citados nos fóruns/comunidade do Moto E4.

---

## Aviso final

Esta ROM é **UNOFFICIAL**.

Instale apenas se você entende os riscos de:

```text
desbloquear bootloader
fazer wipe
instalar recovery custom
instalar ROM custom
perder dados
causar bootloop
```

Faça backup antes.
