# Guia — Desbloquear bootloader, usar TWRP e instalar LineageOS 14.1 no Moto E4 Perry

Este guia cobre o caminho completo para sair de um Moto E4 original/bloqueado até instalar a ROM **LineageOS 14.1 UNOFFICIAL** no **Moto E4 Qualcomm `perry` XT1768**.

ROM usada neste processo:

```text
lineage_perry-ota-4987d1b247.zip
```

Recovery usada:

```text
twrp-3.7.0_9-0-perry.img
```

> Use apenas no **Moto E4 Qualcomm perry**. Não use em Moto E4 MediaTek `woods`, Moto E4 Plus `owens` ou qualquer outro modelo.

---

## 1. O que é cada coisa

### Bootloader

O **bootloader** é o primeiro programa que roda quando o celular liga. Ele decide se o aparelho pode iniciar apenas software oficial ou também imagens customizadas.

Para instalar LineageOS, precisamos desbloquear o bootloader.

### Fastboot

**Fastboot** é o modo de comunicação entre o PC e o bootloader. Ele permite:

```text
verificar aparelho conectado
consultar informações do aparelho
desbloquear bootloader
iniciar TWRP temporariamente
```

### ADB

**ADB** é usado quando o Android ou o TWRP já estão rodando. Ele permite:

```text
enviar arquivos
abrir shell
ver logs
reiniciar para bootloader
```

### TWRP

O **TWRP** é uma recovery customizada. Ele não é a ROM. Ele é a ferramenta usada para instalar a ROM.

Analogia:

```text
TWRP = pendrive de instalação/manutenção
ROM  = sistema operacional que será instalado
```

### ROM

A **ROM** é o sistema Android que será instalado no aparelho. Neste caso:

```text
LineageOS 14.1 / Android 7.1.2
```

Arquivo:

```text
lineage_perry-ota-4987d1b247.zip
```

---

## 2. Preparar o PC

No Pop!_OS/Ubuntu:

```bash
sudo apt update
sudo apt install -y android-tools-adb android-tools-fastboot
```

Teste:

```bash
adb version
fastboot --version
```

Separe os arquivos:

```text
~/Downloads/twrp-3.7.0_9-0-perry.img
~/Downloads/lineage_perry-ota-4987d1b247.zip
```

Se a ROM ainda estiver na pasta do build:

```bash
cp /home/deyvid/android/lineage-14.1-perry/lineage_perry-ota-4987d1b247.zip ~/Downloads/
```

---

## 3. Ativar opções no Moto E4

No Android original:

```text
Configurações > Sobre o telefone > Número da versão
```

Toque várias vezes até ativar as opções de desenvolvedor.

Depois vá em:

```text
Configurações > Sistema > Opções do desenvolvedor
```

Ative:

```text
Desbloqueio de OEM
Depuração USB
```

Conecte o celular no PC e rode:

```bash
adb devices
```

Se aparecer:

```text
unauthorized
```

desbloqueie a tela e aceite a autorização de depuração USB.

---

## 4. Solicitar unlock key no site da Motorola

Acesse:

```text
https://en-us.support.motorola.com/app/standalone/bootloader/unlock-your-device-b
```

Faça login se o site pedir.

Reinicie o celular para bootloader:

```bash
adb reboot bootloader
```

Ou entre manualmente:

```text
Desligue o aparelho
Segure Volume - + Power
```

Confirme no PC:

```bash
fastboot devices
```

Esperado:

```text
ZY2252MSHW    fastboot
```

Agora gere o código de desbloqueio:

```bash
fastboot oem get_unlock_data
```

O retorno vem em várias linhas:

```text
(bootloader) XXXXX
(bootloader) YYYYY
(bootloader) ZZZZZ
```

Copie tudo e transforme em uma única linha, removendo:

```text
(bootloader)
espaços
quebras de linha
```

Cole essa string no site da Motorola, aceite os termos e solicite a unlock key.

A Motorola envia a chave por e-mail.

---

## 5. Desbloquear o bootloader

Com o aparelho em fastboot:

```bash
fastboot oem unlock SUA_CHAVE_RECEBIDA
```

Em alguns casos é necessário repetir o comando para confirmar:

```bash
fastboot oem unlock SUA_CHAVE_RECEBIDA
```

> Isso normalmente apaga os dados do celular.

Confirme:

```bash
fastboot getvar all 2>&1 | grep -Ei "product|sku|securestate|unlocked"
```

Resultado esperado para nosso caso:

```text
product: perry
sku: XT1768
securestate: flashing_unlocked
```

---

## 6. Confirmar se é realmente perry

Antes de instalar qualquer coisa, confirme:

```bash
fastboot getvar all 2>&1 | grep -Ei "product|sku|cid|bootloader|baseband|unlocked|secure|variant"
```

Nosso aparelho:

```text
product: perry
board: perry
sku: XT1768
cpu: MSM8917
securestate: flashing_unlocked
ro.carrier: retus
version-baseband: PERRY_NA_CUST
```

Não prossiga se aparecer:

```text
woods
owens
```

ou outro codinome diferente de `perry`.

---

## 7. Iniciar TWRP temporariamente

Com o aparelho em fastboot:

```bash
cd ~/Downloads
fastboot boot twrp-3.7.0_9-0-perry.img
```

Isso não instala o TWRP permanentemente. Ele roda temporariamente na RAM.

Se aparecer `bad key`, isso pode ser normal em Motorola com bootloader desbloqueado.

---

## 8. Format Data e Wipe

### Format Data

No TWRP:

```text
Wipe > Format Data
```

Ele pede para digitar:

```text
yes
```

Isso apaga:

```text
apps
configurações
armazenamento interno
criptografia antiga
```

Mas normalmente não apaga:

```text
Boot
Modem
Firmware
Persist
EFS
Bootloader
```

### Advanced Wipe recomendado

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

## 9. Enviar a ROM para o celular

Com o TWRP aberto:

```bash
adb devices
```

Esperado:

```text
ZY2252MSHW    recovery
```

Envie a ROM:

```bash
adb push ~/Downloads/lineage_perry-ota-4987d1b247.zip /sdcard/
```

Se `/sdcard` não funcionar:

```bash
adb push ~/Downloads/lineage_perry-ota-4987d1b247.zip /tmp/
```

---

## 10. Instalar a ROM

No TWRP:

```text
Install
```

Selecione:

```text
lineage_perry-ota-4987d1b247.zip
```

Depois:

```text
Swipe to confirm Flash
```

Aguarde terminar.

Para uso como servidor leve, **não instale GApps**.

---

## 11. Primeiro boot

Depois da instalação:

```text
Reboot System
```

Tempo esperado:

```text
5–10 minutos = normal
10–15 minutos = aceitável
20+ minutos = suspeito de bootloop
```

Se aparecer `bad key`, pode ser normal por bootloader desbloqueado.

Verifique pelo PC:

```bash
adb devices
```

Se aparecer:

```text
device
```

o sistema iniciou.

Se aparecer:

```text
unauthorized
```

desbloqueie a tela e aceite a depuração USB.

---

## 12. Se der bootloop

Se ficar mais de 20 minutos na animação:

1. Segure Power para forçar reinício/desligamento.
2. Entre em fastboot com Volume - + Power.
3. Rode:

```bash
cd ~/Downloads
fastboot boot twrp-3.7.0_9-0-perry.img
```

4. Faça Advanced Wipe:

```text
Dalvik / ART Cache
System
Data
Cache
```

5. Reinstale o ZIP.

Não apague:

```text
Firmware
Persist
Vpersist
Modem
EFS
Boot
```

---

## 13. GApps

Para servidor leve:

```text
não instalar GApps
```

Motivos:

```text
menos RAM
menos serviços em segundo plano
menos armazenamento usado
melhor para Termux/SSH/IoT
```

Se quiser usar como smartphone convencional:

```text
Platform: ARM
Android: 7.1
Variant: pico
```

---

## 14. Pós-instalação com Termux

Depois que a ROM iniciar:

1. Conecte ao Wi-Fi.
2. Instale F-Droid.
3. Instale Termux pelo F-Droid.
4. Instale Termux:Boot.

No Termux:

```bash
pkg update && pkg upgrade
pkg install openssh termux-services nano vim git curl wget python nodejs tmux
passwd
termux-wake-lock
sshd
```

Descobrir usuário:

```bash
whoami
```

Descobrir IP:

```bash
ip addr
```

Acessar do PC:

```bash
ssh -p 8022 usuario@ip_do_celular
```

A porta SSH do Termux é:

```text
8022
```

---

## 15. Resumo rápido

```bash
adb devices
adb reboot bootloader
fastboot devices
fastboot oem get_unlock_data
fastboot oem unlock SUA_CHAVE
fastboot getvar all 2>&1 | grep -Ei "product|sku|securestate"
cd ~/Downloads
fastboot boot twrp-3.7.0_9-0-perry.img
adb push ~/Downloads/lineage_perry-ota-4987d1b247.zip /sdcard/
```

No TWRP:

```text
Wipe > Advanced Wipe > Dalvik / ART Cache, System, Data, Cache
Install > lineage_perry-ota-4987d1b247.zip > Swipe
Reboot System
```
