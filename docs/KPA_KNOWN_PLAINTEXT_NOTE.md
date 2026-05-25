# KPA Known Plaintext Note / 已知明文攻击说明

## 中文说明

在 ZIP 已知明文攻击（KPA）中，“明文”并不总是解压后的原始文件内容。

更准确地说，KPA 需要的是 **ZipCrypto 加密前的字节流**。

ZIP 条目的常见处理顺序是：

```text
原始文件内容 -> 压缩 -> ZipCrypto 加密 -> 写入 ZIP 条目
```

因此：

- 如果目标条目是 `ZIP_STORED`，也就是未压缩，那么原始文件内容通常就是加密前明文，可以直接作为 `-kpa` 输入。
- 如果目标条目是 `ZIP_DEFLATED` / `ZIP_BZIP2` / `ZIP_LZMA` 等压缩方式，那么被加密的通常是压缩后的数据流，而不是解压后的原始文件。
- 这种情况下，直接把原始文件作为 `-kpa` 明文可能会失败，并可能出现类似错误：

```text
ciphertext is smaller than plaintext
```

这个报错通常表示：你提供的明文文件长度大于目标 ZIP 条目的加密载荷长度。常见原因是原始文件经过压缩后变小，而 KPA 实际需要的是压缩后的加密前数据流。

## 关于 `--kpa-offset`

`--kpa-offset` 只表示“这段已知字节在加密前数据流中的起始偏移”。

它可以解决：

```text
已知字节不是从偏移 0 开始
```

但它不能解决：

```text
提供的是未压缩原文件，而目标密文对应的是压缩后数据流
```

如果数据层级不一致，单纯调整偏移通常没有意义。

## 推荐使用方式

1. 目标条目为 `ZIP_STORED` 时，可以优先尝试普通原始文件：

```bash
python3 ZipCracker.py target.zip -kpa plain.bin -c inner.bin
```

2. 目标条目为 `ZIP_DEFLATED` 等压缩方式时，优先提供对应的无密码对照 ZIP：

```bash
python3 ZipCracker.py encrypted.zip -kpa plain_reference.zip
```

3. 如果只有部分已知字节，可以使用偏移和附加字节：

```bash
python3 ZipCracker.py secret.zip -kpa part.bin --kpa-offset 78 -x 0 4d5a
```

4. 如果只有常见文件头，可以使用模板：

```bash
python3 ZipCracker.py target.zip --kpa-template png -c image.png
python3 ZipCracker.py target.zip --kpa-template exe -c app.exe
python3 ZipCracker.py target.zip --kpa-template pcapng -c capture.pcapng
python3 ZipCracker.py target.zip --kpa-template zip -c inside.zip
```

## English Note

In a ZIP known-plaintext attack (KPA), “plaintext” does not always mean the original file content after extraction.

More precisely, KPA needs the byte stream **before ZipCrypto encryption**.

A common ZIP entry pipeline is:

```text
original file content -> compression -> ZipCrypto encryption -> ZIP entry payload
```

Therefore:

- If the target entry is `ZIP_STORED`, meaning no compression is used, the original file content can usually be used directly as `-kpa` plaintext.
- If the target entry is `ZIP_DEFLATED`, `ZIP_BZIP2`, `ZIP_LZMA`, or another compressed method, the encrypted data is usually the compressed stream, not the original uncompressed file.
- In that case, passing the original file directly as `-kpa` plaintext may fail with an error such as:

```text
ciphertext is smaller than plaintext
```

This usually means the plaintext file you provided is larger than the encrypted payload of the target ZIP entry. A common reason is that the original file became smaller after compression, while KPA needs the compressed pre-encryption byte stream.

## About `--kpa-offset`

`--kpa-offset` only means “the starting offset of the known bytes inside the pre-encryption byte stream”.

It can solve this case:

```text
known bytes do not start at offset 0
```

But it cannot solve this case:

```text
the provided file is the uncompressed original file, while the ciphertext corresponds to compressed data
```

If the data layer is different, simply changing the offset usually will not help.

## Recommended Usage

1. For `ZIP_STORED` entries, try the original file first:

```bash
python3 ZipCracker.py target.zip -kpa plain.bin -c inner.bin
```

2. For `ZIP_DEFLATED` or other compressed entries, prefer a matching unencrypted reference ZIP:

```bash
python3 ZipCracker.py encrypted.zip -kpa plain_reference.zip
```

3. If only partial known bytes are available, use offset and extra bytes:

```bash
python3 ZipCracker.py secret.zip -kpa part.bin --kpa-offset 78 -x 0 4d5a
```

4. If only common file headers are known, use templates:

```bash
python3 ZipCracker.py target.zip --kpa-template png -c image.png
python3 ZipCracker.py target.zip --kpa-template exe -c app.exe
python3 ZipCracker.py target.zip --kpa-template pcapng -c capture.pcapng
python3 ZipCracker.py target.zip --kpa-template zip -c inside.zip
```
