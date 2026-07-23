# SmartUSBHub 正式产品文档源

本仓库只保存面向客户的正式文档源文件和仍需维护的历史产品资料，不承担官网
页面构建，也不保存可重新生成的临时产物。

## 目录

| 产品 | 可编辑真源 | 说明 |
| --- | --- | --- |
| SmartUSBHub Pro 4CH USB2.0 | [`SmartUSBHub_Pro-4CH_USB2.0/source/`](./SmartUSBHub_Pro-4CH_USB2.0/source/) | 中英文正式版 DOCX |
| SmartUSBHub Pro 7CH USB2.0 | [`SmartUSBHub_Pro-7CH_USB2.0/source/`](./SmartUSBHub_Pro-7CH_USB2.0/source/) | 中英文正式版 DOCX |
| SmartUSBHub Pro 4CH USB3.0 | [`SmartUSBHub_Pro-4CH_USB3.0/`](./SmartUSBHub_Pro-4CH_USB3.0/) | 历史数据手册与 API 资料 |
| SmartUSBSwitch FlexConnect | [`SmartUSBSwitch_FlexConnect/`](./SmartUSBSwitch_FlexConnect/) | 历史规格书与 API 资料 |

## 文档职责

- 使用说明书以各产品 `source/` 中明确命名的正式 DOCX 为唯一可编辑真源。
- 官网产品指南由 `docs/sdk-docs/scripts/docx_to_markdown.py` 从正式 DOCX 自动生成，
  不另行手工维护一份相同正文。
- SDK、通信协议和技术规格 Markdown 位于 `sdk/python-sdk/docs/`。
- Sphinx 配置及官网入口位于独立仓库 `docs/sdk-docs/`。
- 构建临时文件只放 workspace 根目录 `output/` 或各构建仓库的 `build/`。
- 正式 PDF 发布到 `用户资料包/产品说明书/`；跨电脑校对副本同步到
  `/Users/zhang/Desktop/CloudSync/`。
