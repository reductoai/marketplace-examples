# r-1

Reducto r-1 is a page parser. Send one page image. Get the page text back as layout blocks (titles, text, tables).

## Access

r-1 is available in AWS Marketplace through a private offer from Reducto.

1. Get the private offer link for your AWS account from Reducto.
2. Open the link and accept the offer.
3. Open **AWS Marketplace > Manage subscriptions**, find r-1, and select **Configure**.
4. Choose a region that offers `ml.g6e.2xlarge` (for example `us-east-2`), and copy the model package ARN.

## Deploy

| Cloud | Folder |
|---|---|
| AWS SageMaker | [`aws-sagemaker`](aws-sagemaker) |

Supported instance: `ml.g6e.2xlarge` (1x NVIDIA L40S). Real-time endpoints and batch transform both work.

Delete the endpoint when you are done. AWS bills the instance while the endpoint exists.

## Request

`POST /invocations` (SageMaker `InvokeEndpoint`) with `Content-Type: application/json`. Send one page in one of two bodies.

### Page-parse body

```json
{"image_b64": "<base64 PNG or JPEG>"}
```

### Chat-completions body

An OpenAI chat-completions request with one `user` message. The message has one `image_url` part with a `data:` URL of a PNG or JPEG.

```json
{
  "model": "r-1",
  "messages": [
    {
      "role": "user",
      "content": [
        {"type": "image_url", "image_url": {"url": "data:image/png;base64,<base64 PNG>"}}
      ]
    }
  ]
}
```

r-1 ignores `model` and any text part. Remote image URLs, streaming, tools, and `n` > 1 are not supported.

### Optional fields

| Field | Body | Type | Notes |
|---|---|---|---|
| `max_tokens` | both | int | Upper limit on output tokens. |
| `temperature` | both | float | Default `0`. |
| `max_pixels` | both | int | Upper limit on image pixels. The image is resized to fit. Maximum `6291456`. |
| `top_p` | chat | float | Greater than 0, at most 1. |
| `chat_template_kwargs` | chat | object | Only `{"enable_thinking": false}`. |

## Response

The page-parse body returns:

```json
{
  "text": "<page ...>\n\n<block type=\"title\" position=...>\nINVOICE INV-314\n</block>...",
  "finish_reason": "stop",
  "usage": {"prompt_tokens": 644, "completion_tokens": 235, "total_tokens": 879}
}
```

The chat-completions body returns an OpenAI `chat.completion` object. The text is in `choices[0].message.content`.

```json
{
  "id": "chatcmpl-...",
  "object": "chat.completion",
  "created": 1791763200,
  "model": "r-1",
  "choices": [
    {
      "index": 0,
      "message": {"role": "assistant", "content": "<page ...>\n\n<block type=\"title\" position=...>\nINVOICE INV-314\n</block>..."},
      "finish_reason": "stop"
    }
  ],
  "usage": {"prompt_tokens": 644, "completion_tokens": 235, "total_tokens": 879}
}
```

- The text: a `<page>` header, then one `<block>` for each layout region. Each block has a `type` and a `position` box. Tables are HTML.
- `position`: `x0,y0,x1,y1` on a 0-999 grid across the page width and height, not pixels. Pixel x = x0 × width / 999.
- `finish_reason`: `stop` when the page is complete. `length` when `max_tokens` cut the output.

## Errors

| Status | Body `detail` | What to do |
|---|---|---|
| 413 | `request body too large` | Send a smaller image. The limit is 6 MB. |
| 415 | `invalid task input` | Send a valid PNG or JPEG. |
| 422 | `invalid task request` | Fix the request body. |
| 422 | `input could not be processed` | Do not retry. See below. |
| 502, 504 | `inference failed`, `inference timed out` | Retry. |

When r-1 cannot process a page, a real-time call returns HTTP 422 with:

```json
{"detail": "input could not be processed"}
```

- SageMaker shows this error as a 424 `ModelError` with `OriginalStatusCode` 422.
- Do not retry the page. A retry gives the same result.
- In batch transform, the `.out` file of that page holds the same body, and the job continues.

## Batch transform

Put one JSON request body (either body) in each S3 object. Use these settings:

| Setting | Value |
|---|---|
| `ContentType` | `application/json` |
| `SplitType` | `None` |
| `BatchStrategy` | `SingleRecord` |
| `MaxPayloadInMB` | `6` |

Each result goes to `<input>.out`.

## Limits

- Request body: 6 MB maximum.
- Real-time requests must finish within the SageMaker 60 s limit. A page normally takes a few seconds.
- The container has no network access.

## Samples

[`samples`](samples) holds synthetic pages, request bodies, and responses.

| File | Contents |
|---|---|
| `invoice.request.json` | Page-parse request body for `invoice.png`. |
| `invoice.response.json` | Page-parse response for `invoice.png`. |
| `invoice.chat.request.json` | Chat-completions request body for `invoice.png`. |
| `invoice.chat.response.json` | Chat-completions response for `invoice.png`. |
| `text.response.json`, `rotated.response.json` | Page-parse responses for `text.png` and `rotated.png`. |
