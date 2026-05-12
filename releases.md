# Releases

Este arquivo documenta as releases deste projeto e quais arquivos devem ser anexados na aba **GitHub Releases**.

Os binários grandes, como ROM `.zip` e TWRP `.img`, **não devem ser commitados diretamente no Git**. Eles devem ser anexados à release.

---

## v14.1-perry-20260512

### Título sugerido

```text
LineageOS 14.1 UNOFFICIAL para Moto E4 Qualcomm perry XT1768
```

### Arquivos da release

Anexar na release:

```text
lineage_perry-ota-4987d1b247.zip
twrp-3.7.0_9-0-perry.img
lineage_perry-ota-4987d1b247.sha256
```

Opcional, se gerar:

```text
twrp-3.7.0_9-0-perry.sha256
```

---

## Descrição sugerida para a release

```md
# LineageOS 14.1 UNOFFICIAL — Moto E4 Qualcomm perry XT1768

Build não oficial da LineageOS 14.1, baseada no Android 7.1.2, para o Moto E4 Qualcomm `perry`.

## Aparelho alvo

- Moto E4 Qualcomm
- Codinome: `perry`
- Modelo testado: XT1768
- Plataforma: Qualcomm MSM8917
- Canal original visto no aparelho: retus / retail.en.US
- Baseband vista: PERRY_NA_CUST

Não use em:

- Moto E4 MediaTek `woods`
- Moto E4 Plus `owens`
- outros modelos/codinomes

## Arquivos

- ROM: `lineage_perry-ota-4987d1b247.zip`
- Recovery: `twrp-3.7.0_9-0-perry.img`
- Checksum: `lineage_perry-ota-4987d1b247.sha256`

## Instalação resumida

1. Desbloqueie o bootloader pelo site oficial da Motorola.
2. Entre em fastboot.
3. Inicie o TWRP temporariamente:

   ```bash
   fastboot boot twrp-3.7.0_9-0-perry.img
   ```

4. No TWRP, faça Advanced Wipe marcando apenas:

   - Dalvik / ART Cache
   - System
   - Data
   - Cache

5. Envie a ROM:

   ```bash
   adb push lineage_perry-ota-4987d1b247.zip /sdcard/
   ```

6. Instale pelo TWRP:

   ```text
   Install > lineage_perry-ota-4987d1b247.zip > Swipe
   ```

7. Reboot System.

O primeiro boot pode demorar de 5 a 10 minutos.

## GApps

Para uso como servidor leve, Termux, SSH, MQTT e IoT, recomenda-se não instalar GApps.

Se quiser usar como smartphone convencional, use OpenGApps:

- Platform: ARM
- Android: 7.1
- Variant: pico

## Aviso

Use por sua conta e risco.

Não sou responsável por bootloop, perda de dados, brick, cartão SD morto, alarme que não tocou ou qualquer dano.

Faça backup antes.
```

---

## Checksum

Gerar checksum da ROM:

```bash
cd ~/Downloads
sha256sum lineage_perry-ota-4987d1b247.zip > lineage_perry-ota-4987d1b247.sha256
```

Verificar checksum:

```bash
sha256sum -c lineage_perry-ota-4987d1b247.sha256
```

Gerar checksum do TWRP:

```bash
sha256sum twrp-3.7.0_9-0-perry.img > twrp-3.7.0_9-0-perry.sha256
```

---

## Notas da build

Resumo técnico do que foi usado no build:

```text
Base: LineageOS cm-14.1
Android: 7.1.2
Device: perry
Modelo testado: XT1768
Kernel: android_kernel_motorola_msm8937
Vendor: android_vendor_motorola_perry
Build final: lineage_perry-ota-4987d1b247.zip
```

Correções aplicadas durante o processo:

```text
Git LFS para WebView prebuilt
Jack server com Java 8
TLSv1/TLSv1.1 reabilitado no java.security para evitar erro SSL do Jack
export USER=root para evitar USER: unbound variable
TARGET_KERNEL_CONFIG := perry_defconfig também em qcom318-32/BoardConfigCommon.mk
```

---

## Histórico

### v14.1-perry-20260512

```text
Primeira release preservada do build local.
ZIP gerado com sucesso.
Tamanho aproximado: 367 MB.
Arquivo: lineage_perry-ota-4987d1b247.zip
```
