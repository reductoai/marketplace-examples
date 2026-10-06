# r-1

Reducto r-1 is a page parser. Send one page image. Get the page text back as layout blocks (titles, text, tables).

## Deploy

| Cloud | Folder |
|---|---|
| AWS SageMaker | [`aws-sagemaker`](aws-sagemaker) |

Supported instance: `ml.g6e.2xlarge` (1x NVIDIA L40S). Real-time endpoints and batch transform both work.

## Request

`POST /invocations` with `Content-Type: application/json`.

```json
{"image_b64": "<base64 PNG or JPEG>"}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `image_b64` | string | yes | One page, PNG or JPEG, base64. |
| `max_pixels` | int | no | Upper limit on image pixels. |
| `max_tokens` | int | no | Upper limit on output tokens. |
| `temperature` | float | no | Default `0`. |

## Response

```json
{
  "text": "<page ...>\n\n<block type=\"title\" position=...>\nINVOICE INV-314\n</block>...",
  "finish_reason": "stop",
  "usage": {"prompt_tokens": 644, "completion_tokens": 235, "total_tokens": 879}
}
```

- `text`: a `<page>` header, then one `<block>` for each layout region. Each block has a `type` and a `position` box. Tables are HTML.
- `position`: `x0,y0,x1,y1` on a 0-999 grid across the page width and height, not pixels. Pixel x = x0 × width / 999.
- `finish_reason`: `stop` when the page is complete. `length` when `max_tokens` cut the output.

## Limits

- Request body: 6 MB maximum.
- Real-time requests must finish within the SageMaker 60 s limit. A page normally takes a few seconds.
- The container has no network access.

## Samples

[`samples`](samples) holds synthetic pages and real responses. `invoice.request.json` is a full request body for `invoice.png`.
