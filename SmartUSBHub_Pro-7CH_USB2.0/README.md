# SmartUSBHub Pro 7CH USB2.0 产品说明书

本目录的 `source/` 子目录保存经确认的正式 Word 文档源文件：

- `source/SmartUSBHub_Pro_7CH_USB2.0_使用说明书_正式版_v1.0.docx`
- `source/SmartUSBHub_Pro_7CH_USB2.0_User_Manual_Released_v1.0_EN.docx`

它们是 7CH 中英文使用指南的唯一内容真源。`docs/sdk-docs` 构建前会将 DOCX
转换为 `sdk/python-sdk/docs/products/usb2_7p/` 中的 Markdown，再由
Sphinx 文档中心生成网页。不要独立手工维护 Markdown 使用指南。

构建过程生成的 DOCX、PDF 和逐页渲染图统一放在 workspace 根目录的
`output/`。该目录仅存放可重新生成的临时产物，正式发布后可以全部清空。
