## 概述

FastAPI 是一个现代、快速（高性能）的Web框架，用于构建APIs。该项目基于 Fastapi 构建了 Fooocus 的 Rest 接口。

## 功能特性

- 完整保留了 Fooocus 的代码，也即 Fooocus 的所有功能以及使用不受影响
- 可以同时启动 API 以及 WebUI，或者选择不启动 WebUI
- 使用 X-API-KEY 进行接口认证
- 完整的 Fooocus 支持
- all-in-one 接口
- 使用 URL 提供 INPUT 图像
- 接口提供了流式输出、二进制图像以及异步任务三种方式
- 持久化任务历史
- 增强的任务历史管理
- 任务查询功能
- WebHook 支持

## 安装

在 Fooocus 基础上多了数个依赖库，因此可以按照 Fooocus 的方式进行安装。

## 启动

和 Fooocus 相同的启动方式，使用 `--nowebui` 参数可以启动 API 而不启动 WebUI。

默认的 API 端口是 WebUI 端口加 1，即 7866，使用 `--port` 修改 WebUI 端口将同时修改 API 端口。

同时启动 WebUI 和 API 示例：

```shell
python launch.py --listen 0.0.0.0 --port 7865
```

仅启动 API 示例：

```shell
python launch.py --listen 0.0.0.0 --port 7865 --nowebui
```

## 安全

- 使用API密钥进行认证，密钥通过请求头`X-API-KEY`传递。

使用 `--apikey` 指定接口认证密钥

## EndPoints

### 生成

`POST /v1/engine/generate/`
- **摘要**: 生成API路由
- **请求体**: 必须，JSON格式，基于`CommonRequest`模型。
- **响应**:
  - 200: 成功响应，返回生成结果。

说明：

请求参数模型 `CommonRequest` 包含 WebUI 中的所有参数，但有些参数你需要在使用中特别注意，包含以下几类：

在 API 中无效的参数，这部分参数包含：

- `input_image_checkbox`, 该参数总是被设置为 True
- `inpaint_mask_upload_checkbox`, 该参数总是被设置为 True
- `invert_mask_checkbox`, 该参数总是被设置为 false
- `current_tab`, 该参数会检查参数中的图像信息并自动设置，检查顺序为 'ip' -> 'uov' -> 'inpaint'

不建议设置的参数：

- `generate_image_grid`, 不清楚该参数的作用，建议保持 false

以下参数需要根据使用场景进行设置：

- `mixing_image_prompt_and_vary_upscale`
- `mixing_image_prompt_and_inpaint`

此外，还包含部分 API 专用的参数：

- `preset`, 可以使用该参数指定预设, 其优先级高于全局默认, 低于传递参数, 但如果传递参数等于默认值, 则使用预设参数
- `stream_output`, 设定为 true 后会返回流式输出
- `require_base64`, 该参数目前没有作用
- `async_process`, 是否异步任务
- `webhook_url`, Webhook 地址，如果设置了该参数，则任务完成后会发送 POST 请求到该地址

> `stream_output` 的优先级高于 `async_process`, 即同时为 true 的时候, 返回流式输出。当全部为 false 时, 返回二进制图像, 并设置 image_number 为 1


### 获取任务
`GET /tasks`
- **标签**: Query
- **摘要**: 获取所有任务
- **描述**: 根据类型过滤任务，并支持分页和时间过滤。
- **参数**:
  - `query` (string, 默认值: "all"): 任务类型，可以是"all", "history", 或 "current"。
  - `page` (integer, 默认值: 0): 返回的页码，用于历史和待办任务。
  - `page_size` (integer, 默认值: 10): 每页返回的任务数量。
  - `start_at` (string): 过滤任务的开始时间, 仅对 history 生效。
  - `end_at` (string, 默认值为 ISO8601 格式的时间，比如: "2024-06-30T17:57:07.045812"): 过滤任务的结束时间, 仅对 history 生效
  - `action` (string): 删除操作专用, 仅对 history 生效, 会删除数据库中记录以及生成的图片。
- **响应**:
  - 200: 成功响应，返回任务列表。
  - 422: 验证错误。

### 获取特定任务
`GET /tasks/{task_id}`
- **标签**: Query
- **摘要**: 通过ID获取特定任务
- **参数**:
  - `task_id` (string): 任务的ID。
- **响应**:
  - 200: 成功响应，返回特定任务的详情。
  - 422: 验证错误。

### 获取特定输出
`GET /outputs/{data}/{file_name}`
- **标签**: Query
- **摘要**: 通过ID获取特定输出
- **参数**:
  - `data` (string): 日期，生成的图片以天为单位创建的文件夹进行归类, 该部分即生成日期。
  - `file_name` (string): 文件的名称。
- **响应**:
  - 200: 成功响应，返回特定输出的内容。
  - 422: 验证错误。

### 生成API路由
`POST /v1/engine/generate/`
- **标签**: GenerateV1
- **摘要**: 生成API路由
- **请求体**: 必须，JSON格式，基于`CommonRequest`模型。
- **响应**:
  - 200: 成功响应，返回生成结果。
  - 422: 验证错误。

### 描述图像
`POST /v1/tools/describe-image`
- **标签**: GenerateV1
- **摘要**: 描述图像，从图像获取标签
- **描述**: 根据提供的图像和图像类型（照片或动漫），返回描述。
- **参数**:
  - `image_type` (string, 默认值: "Photo"): 图像类型。
- **请求体**: 必须，`multipart/form-data`格式，包含图像文件。
- **响应**:
  - 200: 成功响应，返回`DescribeImageResponse`模型。
  - 422: 验证错误。

### 根端点
`GET /`
- **标签**: Query
- **摘要**: 根端点
- **响应**: 自动跳转到 `/docs`

## 组件

### 模型

#### CommonRequest
- 属性:
  - `prompt` (string): 生成图像的提示。
  - `negative_prompt` (string): 过滤不需要的内容的提示。
  - `style_selections` (array): 生成图像的风格选择。
  - `performance_selection` (Performance): 性能选择，默认 `Speed`，可选 `Quality` `Speed` `Extreme Speed` `Lightning` `Hyper-SD`
  - `aspect_ratios_selection` (string): 生成图像的宽高比选择。默认 1152*896
  - `image_number` (int): 生成图像的数量。默认 1，范围 1-32
  - `output_format` (string): 生成图像的输出格式，默认 `png`，可选 `jpg` `webp`
  - `image_seed` (int): 生成图像的种子，默认 -1 即随机
  - `read_wildcards_in_order` (bool): 是否读取提示中的通配符，默认 false
  - `sharpness` (float): 生成图像的锐度，默认 2.0，范围 0.0-30.0
  - `guidance_scale` (float): 生成图像的指导比例，默认 4，范围 1.0-30.0
  - `base_model_name` (string): 基础模型名称，当前默认 `juggernautXL_v8Rundiffusion.safetensors`
  - `refiner_model_name` (string): refiner 模型名称，当前默认为 None
  - `refiner_switch` (float): refiner 切换时机，默认 0.5，范围0.1-1.0
  - `loras` (Lora): 使用的 lora 列表，默认只使用一个 `sd_xl_offset_example-lora_1.0.safetensors`，格式参考 [Lora](#lora)
  - `input_image_checkbox` (bool): 是否使用输入图像，默认 false，该参数无效，程序会总是将它设置为 True
  - `current_tab` (string): 当前激活的标签页，默认 `uov`，可选 `uov` `inpaint` `outpaint`，你无需传递该参数。
  - `uov_method` (string): 生成图像的 UOV 方法，默认 `disable`，可选项参考 [UpscaleOrVaryMethod](#upscaleorvarymethod)
  - `uov_input_image` (string): Upscale or vary 输入图像的 URL 或者 Base64 格式的图片，默认 "None"
  - `outpaint_selections` (array): Outpaint 方向列表，比如 ["Left", "Right", "Top", "Bottom"]
  - `inpaint_input_image` (string): Inpaint 输入图像的 URL 或者 Base64 格式的图片
  - `inpaint_additional_prompt` (string): Inpaint 额外提示，默认 "None"
  - `inpaint_mask_image_upload` (string): Inpaint 遮罩图像的 URL 或者 Base64 格式的图片
  - `inpaint_mask_upload_checkbox` (bool): 默认为 false，但程序总是将它设置为 True
  - `disable_preview` (bool): 是否禁用预览，默认 false
  - `disable_intermediate_results` (bool): 是否禁用中间结果，默认 false
  - `disable_seed_increment` (bool): 是否禁用种子递增，默认 false
  - `black_out_nsfw` (bool): 是否将 NSFW 内容替换为黑色，默认 false
  - `adm_scaler_positive` (float): The scaler multiplied to positive ADM (use 1.0 to disable). 默认 1.5，范围 0.0-3.0
  - `adm_scaler_negative` (float): The scaler multiplied to negative ADM (use 1.0 to disable). 默认 1.5，范围 0.0-3.0
  - `adm_scaler_end` (float): ADM Guidance End At Step，默认 0.8，范围 0.0-1.0
  - `adaptive_cfg` (float): Adaptive cfg，默认 7，范围 1.0-30.0
  - `clip_skip` (float): 生成图像的 clip skip，默认 2，范围 1-12
  - `sampler_name` (string): 采样器名称，默认 dpmpp_2m_sde_gpu
  - `scheduler_name` (string): 调度器名称，默认 karras
  - `vae_name` (string): VAE 名称，默认 Default (model)
  - `overwrite_step` (int): 覆盖 Performance 中的默认步数，默认 -1 不覆盖
  - `overwrite_switch` (float): 覆盖 refiner_switch，默认 -1 不覆盖
  - `overwrite_width` (int): 覆盖 aspect_ratios_selection 中的 width，默认 -1 不覆盖，范围 -1-2048
  - `overwrite_height` (int): 覆盖 aspect_ratios_selection 中的 height，默认 -1 不覆盖，范围 -1-2048
  - `overwrite_vary_strength` (float): 覆盖 vary_strength，默认 -1 不覆盖，范围 0.0-1.0
  - `overwrite_upscale_strength` (float): 覆盖 upscale_strength，默认 -1 不覆盖，范围 0.0-1.0
  - `mixing_image_prompt_and_vary_upscale` (bool): 同时使用 uov 和 imagePrompt，默认 false，该选项不会自动判断，如需使用请指定
  - `mixing_image_prompt_and_inpaint` (bool): 同时使用 uov 和 inpaint，默认 false，该选项不会自动判断，如需使用请指定
  - `debugging_cn_preprocessor` (bool): 是否启用 ControlNet 预处理调试模式，默认 false
  - `skipping_cn_preprocessor` (bool): 跳过 ControlNet 预处理，默认 false
  - `canny_low_threshold` (int): 默认 64，范围 1-255
  - `canny_high_threshold` (int): 默认 128，范围 1-255
  - `refiner_swap_method` (string): 默认 joint
  - `controlnet_softness` (float): 默认 0.25，范围 0.0-1.0
  - `freeu_enabled` (bool): 是否启用 freeu，默认 false
  - `freeu_b1` (float): 默认 1.01
  - `freeu_b2` (float): 默认 1.02
  - `freeu_s1` (float): 默认 0.99
  - `freeu_s2` (float): 默认 0.95
  - `debugging_inpaint_preprocessor` (bool): 是否启用 Inpaint 预处理调试模式，默认 false
  - `inpaint_disable_initial_latent` (bool): 默认 false
  - `inpaint_engine` (string): 默认 v2.6
  - `inpaint_strength` (float): 默认 1.0，范围 0.0-1.0
  - `inpaint_respective_field` (float): 默认0.618，范围 0.0-1.0
  - `invert_mask_checkbox` (bool): 默认 false，该参数总是被设置为 false
  - `inpaint_erode_or_dilate` (int): 默认 0，范围 -64-64
  - `save_metadata_to_images` (bool): 是否保存元数据到图像中，默认 true
  - `metadata_scheme` (string): 默认 foocus，可选 fooocus, a111
  - `controlnet_image` (ImagePrompt): ImagePrompt
  - `generate_image_grid` (bool): 默认 false，在 API 中应该没有作用，建议保持默认
  - `preset` (string): 预设名称，默认 initial
  - `stream_output` (bool): 是否流式输出，默认 false
  - `require_base64` (bool): 暂时无用
  - `async_process` (bool): 是否异步处理，默认 false
  - `webhook_url` (string): Webhook URL，默认 ""


#### Lora

- 属性：
  - `enabled` (bool): 是否启用 Lora，默认 false
  - `model_name` (string): Lora 名称，默认 None
  - `weight` (float): Lora 权重，默认 0.5，范围 -2-2

#### UpscaleOrVaryMethod

- 属性：
  - "Disabled"
  - "Vary (Subtle)"
  - "Vary (Strong)"
  - "Upscale (1.5x)"
  - "Upscale (2x)"
  - "Upscale (Fast 2x)"

#### ImagePrompt

- 属性：
  - `cn_img` (str): ImageUrl 或者 base64 格式图片
  - `cn_stop` (float): 默认 0.6，范围 0-1
  - `cn_weight` (float): 默认 0.5，范围 0-2
  - `cn_type` (string): 默认 ImagePrompt，可选 FaceSwap，PyraCanny，CPDS

#### DescribeImageResponse

- 属性:
  - `describe` (string): 图像描述。

#### RecordResponse
- 属性：
  - `id` (int): 数据库 ID，对于返回来说是无用的。
  - `task_id` (str): 任务 ID，任务 ID，使用 uuid.uuid4().hex 生成。
  - `req_params` (CommonRequest): 请求参数，对于图片，会转换为 URL。
  - `in_queue_mills` (int): 记录进入队列的时间（毫秒）。
  - `start_mills` (int): 记录开始处理的时间（毫秒）。
  - `finish_mills` (int): 记录完成处理的时间（毫秒）。
  - `task_status` (str): 任务状态。
  - `progress` (float): 任务进度。
  - `preview` (str): 任务预览。
  - `webhook_url` (str): 任务 Webhook URL。
  - `result` (List): 任务结果。
