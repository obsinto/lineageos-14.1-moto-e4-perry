# Guia — Build LineageOS 14.1 para Moto E4 Qualcomm Perry XT1768

Este guia documenta o processo de build da **LineageOS 14.1** para o **Moto E4 Qualcomm `perry`**, incluindo ambiente Docker, sincronização do source, device tree, kernel, vendor blobs, problemas encontrados e soluções.

ZIP gerado:

```text
lineage_perry-ota-4987d1b247.zip
```

---

## 1. Por que buildamos a ROM

Queríamos baixar:

```text
lineage-14.1-20210421-UNOFFICIAL-perry.zip
```

Mas o AndroidFileHost retornava:

```text
OOPS!
no mirrors found
```

Como o ZIP pronto não estava mais disponível, decidimos compilar a ROM manualmente.

---

## 2. Identificação do aparelho

Com o Moto E4 em fastboot:

```bash
fastboot getvar all 2>&1 | grep -Ei "product|sku|cid|bootloader|baseband|unlocked|secure|variant"
```

Resultado relevante:

```text
product: perry
board: perry
sku: XT1768
cpu: MSM8917
ram: 2GB
storage-type: emmc
securestate: flashing_unlocked
ro.carrier: retus
version-baseband: PERRY_NA_CUST
ro.build.fingerprint: motorola/perry_f/perry_f:7.1.1/NPQS26.69-64-23/30:user/release-keys
```

Conclusão:

```text
Moto E4 Qualcomm XT1768
Codinome perry
Plataforma MSM8917
Canal retus / retail.en.US
```

---

## 3. Ambiente Docker

Criamos a pasta:

```bash
mkdir -p ~/android/lineage-14.1-perry
cd ~/android/lineage-14.1-perry
```

Subimos o container:

```bash
docker run -it --name lineage141-perry   -v "$PWD":/android   ubuntu:18.04 bash
```

Mais tarde, para acesso USB:

```bash
docker run -it --name lineage141-perry   --privileged   -v /dev/bus/usb:/dev/bus/usb   -v "$PWD":/android   ubuntu:18.04 bash
```

---

## 4. Dependências do container

Dentro do container:

```bash
apt update

apt install -y bc bison build-essential ccache curl flex g++-multilib gcc-multilib   git gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev lib32z1-dev   liblz4-tool libncurses5-dev libxml2 libxml2-utils   lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev   openjdk-8-jdk python python3 git-lfs android-tools-adb android-tools-fastboot
```

Importância:

```text
openjdk-8-jdk = exigido pelo Android 7/Lineage 14.1
python        = scripts antigos do Android
git-lfs       = necessário para WebView prebuilt
adb/fastboot  = comunicação com aparelho
ccache        = acelera rebuilds
```

---

## 5. Instalar repo

```bash
mkdir -p ~/bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
export PATH=~/bin:$PATH
```

Configurar Git:

```bash
git config --global user.name "N F Santana"
git config --global user.email "contato@agilytech.com"
```

---

## 6. Sincronizar LineageOS 14.1

```bash
cd /android
mkdir -p lineage
cd lineage

repo init -u https://github.com/LineageOS/android.git -b cm-14.1
repo sync -c --force-sync --no-clone-bundle --no-tags --retry-fetches=10 -j4
```

Problemas encontrados:

```text
RPC failed; curl 56 GnuTLS recv error
fatal: early EOF
fatal: index-pack failed
```

Solução:

```bash
repo sync -c --force-sync --no-clone-bundle --no-tags --retry-fetches=10 -j1
```

Problema com Git LFS:

```text
git-lfs: not found
```

Solução:

```bash
apt install -y git-lfs
git lfs install
repo sync -c --force-sync --no-clone-bundle --no-tags --retry-fetches=10 -j4
```

---

## 7. Device tree do perry

Clonamos:

```bash
git clone https://github.com/Bcrichster/android_device_motorola_perry.git /tmp/perry-src
```

Copiamos:

```bash
mkdir -p device/motorola

cp -a /tmp/perry-src/perry device/motorola/perry
cp -a /tmp/perry-src/msm8937-common device/motorola/msm8937-common
cp -a /tmp/perry-src/qcom318-32 device/motorola/qcom318-32
```

Adicionamos o manifest extra:

```bash
mkdir -p .repo/local_manifests
cp /tmp/perry-src/roomservice.xml .repo/local_manifests/roomservice.xml

repo sync -c --force-sync --no-clone-bundle --no-tags --retry-fetches=10 -j4
```

---

## 8. Kernel

O device tree indicava:

```makefile
TARGET_KERNEL_CONFIG := perry_defconfig
```

O common indicava:

```makefile
TARGET_KERNEL_SOURCE := kernel/motorola/msm8937
```

Clonamos:

```bash
mkdir -p kernel/motorola

git clone https://github.com/Bcrichster/android_kernel_motorola_msm8937.git   kernel/motorola/msm8937
```

Confirmamos:

```bash
find kernel/motorola/msm8937 -name "perry_defconfig"
```

Resultado:

```text
kernel/motorola/msm8937/arch/arm/configs/perry_defconfig
kernel/motorola/msm8937/arch/arm64/configs/perry_defconfig
```

---

## 9. Vendor blobs

Inicialmente tínhamos apenas:

```text
vendor/motorola/qcom318-32
```

Tentamos TheMuppets:

```bash
git ls-remote --heads https://github.com/TheMuppets/proprietary_vendor_motorola.git | grep cm-14.1
```

A branch existia, mas não havia `perry`.

Testamos o vendor do mantenedor:

```bash
git ls-remote https://github.com/Bcrichster/android_vendor_motorola_perry.git
```

Resultado:

```text
refs/heads/cm-14.1
```

Clonamos:

```bash
cd /home/deyvid/android/lineage-14.1-perry/lineage

git clone   https://github.com/Bcrichster/android_vendor_motorola_perry.git   -b cm-14.1   vendor/motorola/perry
```

Confirmamos:

```bash
ls vendor/motorola/perry
```

Resultado:

```text
Android.mk
BoardConfigVendor.mk
perry-vendor.mk
proprietary
```

Estado:

```text
vendor/motorola/perry
vendor/motorola/qcom318-32
```

---

## 10. Ajuste de kernel config

Durante o build apareceu:

```text
Kernel source found, but no configuration was defined
```

Confirmamos:

```bash
grep -R "TARGET_KERNEL_CONFIG" -n   device/motorola/perry   device/motorola/qcom318-32
```

O `TARGET_KERNEL_CONFIG` só aparecia em `device/motorola/perry/BoardConfig.mk`.

Adicionamos também no common:

```bash
sed -i '/TARGET_KERNEL_SOURCE := kernel\/motorola\/msm8937/a TARGET_KERNEL_CONFIG := perry_defconfig'   device/motorola/qcom318-32/BoardConfigCommon.mk
```

Depois:

```bash
grep -R "TARGET_KERNEL_CONFIG" -n   device/motorola/perry   device/motorola/qcom318-32
```

Resultado esperado:

```text
device/motorola/perry/BoardConfig.mk:24:TARGET_KERNEL_CONFIG := perry_defconfig
device/motorola/qcom318-32/BoardConfigCommon.mk:47:TARGET_KERNEL_CONFIG := perry_defconfig
```

---

## 11. Configurar ambiente para build

Dentro do container:

```bash
cd /android/lineage

export USER=root
export LOGNAME=root
export HOME=/root

echo 'export USER=root' >> /root/.bashrc
echo 'export LOGNAME=root' >> /root/.bashrc
echo 'export HOME=/root' >> /root/.bashrc
```

Essas variáveis foram necessárias porque o Jack falhava com:

```text
USER: unbound variable
```

---

## 12. Corrigir TLS do Java para Jack

Erro encontrado:

```text
Communication error with Jack server (35)
SSL error when connecting to the Jack server
```

Corrigimos o Java 8:

```bash
cp /etc/java-8-openjdk/security/java.security /etc/java-8-openjdk/security/java.security.bak

sed -i 's/, TLSv1, TLSv1.1//g' /etc/java-8-openjdk/security/java.security
sed -i 's/TLSv1, TLSv1.1, //g' /etc/java-8-openjdk/security/java.security
sed -i 's/TLSv1, TLSv1.1//g' /etc/java-8-openjdk/security/java.security
```

Verificação:

```bash
grep "jdk.tls.disabledAlgorithms" /etc/java-8-openjdk/security/java.security
```

A linha não deve bloquear `TLSv1` nem `TLSv1.1`.

---

## 13. Subir Jack

```bash
cd /android/lineage

export JACK_SERVER_VM_ARGUMENTS="-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx4g"
export ANDROID_JACK_VM_ARGS="-Dfile.encoding=UTF-8 -Xmx4g"

prebuilts/sdk/tools/jack-admin kill-server || true
pkill -f com.android.jack.launcher.ServerLauncher || true
pkill -f jack-admin || true

prebuilts/sdk/tools/jack-admin start-server > /tmp/jack-start.log 2>&1 &
sleep 8

prebuilts/sdk/tools/jack-admin list-server
```

Se listar o processo Java do Jack, prossiga.

---

## 14. Build

```bash
cd /android/lineage

source build/envsetup.sh
breakfast perry
brunch perry 2>&1 | tee /android/lineage/build-perry.log
```

Monitoramento no host:

```bash
watch -n 10 'df -h /; free -h'
```

---

## 15. Problema com WebView e Git LFS

O build chegou em 51% e falhou no WebView:

```text
target Prebuilt: webview
java.util.zip.ZipException: error in opening zip file
```

Verificamos:

```bash
ls -lh external/chromium-webview/prebuilt/arm/webview.apk
file external/chromium-webview/prebuilt/arm/webview.apk
head -20 external/chromium-webview/prebuilt/arm/webview.apk
```

Resultado:

```text
133 bytes
ASCII text
version https://git-lfs.github.com/spec/v1
```

Ou seja, era só um ponteiro Git LFS, não o APK real.

Solução:

```bash
git config --global --add safe.directory /android/lineage/external/chromium-webview/prebuilt/arm

cd /android/lineage/external/chromium-webview/prebuilt/arm
git lfs pull
```

Verificação:

```bash
ls -lh webview.apk
file webview.apk
unzip -t webview.apk
```

Depois limpar intermediários:

```bash
cd /android/lineage
rm -rf out/target/product/perry/obj/APPS/webview_intermediates
rm -rf out/target/product/perry/system/app/webview
```

Retomar:

```bash
source build/envsetup.sh
breakfast perry
brunch perry 2>&1 | tee /android/lineage/build-perry.log
```

---

## 16. Resultado final

O build chegou em 99%:

```text
[ 99% ] Install system fs image
```

E gerou:

```text
/android/lineage/out/target/product/perry/lineage_perry-ota-4987d1b247.zip
```

Tamanho aproximado:

```text
367 MB
```

Copiamos para fora do container:

```bash
cp /android/lineage/out/target/product/perry/lineage_perry-ota-4987d1b247.zip /android/
```

No host:

```text
/home/deyvid/android/lineage-14.1-perry/lineage_perry-ota-4987d1b247.zip
```

Para mover para Downloads:

```bash
cp /home/deyvid/android/lineage-14.1-perry/lineage_perry-ota-4987d1b247.zip ~/Downloads/
```

Checksum:

```bash
cd ~/Downloads
sha256sum lineage_perry-ota-4987d1b247.zip > lineage_perry-ota-4987d1b247.sha256
```

---

## 17. Resumo do estado final

```text
✅ LineageOS 14.1 sincronizado
✅ device tree perry
✅ kernel msm8937
✅ vendor perry
✅ vendor qcom318-32
✅ Jack corrigido
✅ WebView LFS corrigido
✅ build finalizado
✅ ZIP gerado
```

Arquivo final:

```text
lineage_perry-ota-4987d1b247.zip
```
