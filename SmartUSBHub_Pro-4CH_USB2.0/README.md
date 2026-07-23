# SmartUSBHub Pro 4CH USB2.0 产品说明书

本目录的 `source/` 子目录保存可编辑的正式 Word 文档源文件，也是官网
4CH 产品说明书的唯一内容真源：

- `source/SmartUSBHub_Pro_4CH_USB2.0_使用说明书_正式版_v1.0.docx`
- `source/SmartUSBHub_Pro_4CH_USB2.0_User_Manual_Released_v1.0_EN.docx`

`docs/sdk-docs` 构建时固定读取上述经确认的正式发布文件，并直接从 DOCX
生成网页内容。目录中其他评审或迭代文件不得通过“版本号最大”规则自动发布。
网页不再维护另一份 4CH Markdown 使用指南。

构建过程生成的 DOCX、PDF 和逐页渲染图统一放在 workspace 根目录的
`output/`。该目录仅存放可重新生成的临时产物，正式发布后可以全部清空。

面向用户交付的 PDF 发布副本位于 workspace 根目录的：

`用户资料包/产品说明书/SmartUSBHub Pro 4CH USB2.0/`

系统框图的真源为：

`scripts/manual/assets/system-block-diagram-en.svg`

生成脚本会在每次构建时重新渲染该 SVG，并将结果嵌入中英文 DOCX。
