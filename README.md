# Exercise #9 — Product Description Generator

A Python CLI that sends one or more product photos and optional facts to an OpenAI multimodal model, then prints a product description and short marketing slogans.

## Setup

Requires Python 3.10+ and an OpenAI API key with model access and available API credit.

```bash
python -m venv .venv
source .venv/bin/activate
# Windows PowerShell: .venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
export OPENAI_API_KEY="your-api-key"
# Windows PowerShell: $env:OPENAI_API_KEY="your-api-key"
```

Do not commit your API key. The program reads environment variables; it does not load `.env` files.

## Run with your product photos

Put your actual photos beside the script or use their full paths:

```bash
python product_marketing.py \
  --images front.jpg side.jpg packaging.jpg \
  --info "Stainless-steel insulated bottle, 750 ml, keeps drinks cold for 24 hours" \
  --tone professional \
  --slogans 3
```

Replace the example filenames and facts with your own. Only supply specifications you have confirmed. For multiple products, identify which facts belong to each product.

Images alone also work:

```bash
python product_marketing.py --images front.jpg
```

`--images` is required and accepts multiple local JPEG, PNG, or static WebP images. Each file is limited to 10 MiB, with a combined limit of 20 MiB. `--info` is optional. `--tone` accepts `professional`, `friendly`, or `enthusiastic`. `--slogans` accepts 1–10 (default 3). `--model` overrides the default `gpt-4.1-mini`; `OPENAI_MODEL` can also set it. Choose a model supporting image inputs and the Responses API.

## Output

The prompt requests these separate sections for each distinct product:

- **Visible in Images:** observations tied to image numbers.
- **User-Supplied Facts:** supplied claims, marked as not independently verified.
- **Uncertainties / Conflicts:** unreadable labels, missing details, and disagreements.
- **Product Description:** clear marketing prose using supported information.
- **Marketing Slogans:** the requested number of short slogans.

The model is instructed not to infer materials, dimensions, capacity, technical specifications, durability, or performance from appearance. Legible packaging claims are attributed to the packaging rather than treated as verified facts. Conflicting claims must be flagged and excluded from marketing text. Image and user text are treated as evidence, not instructions. These are model instructions, not a factual verification system: review all generated claims before publishing.

## Tests

```bash
python -m unittest -v
```

Seven offline tests cover multi-image input, invalid/missing files, request construction and output extraction, API failures, missing credentials, argument validation, and the full CLI flow to terminal output. They mock the API and make no paid requests. A live model run is not part of the offline test suite.

## How it works

`argparse` reads options; Pillow validates images; the script base64-encodes each image and combines them with the product facts in one request. Python's standard HTTP library calls the OpenAI Responses API. Generated text is printed to stdout; errors go to stderr with exit code 1 (argument errors use code 2).

Images and supplied text are transmitted to OpenAI, and API usage may incur charges. Requests set `store: false`; this is not a claim of zero data retention.

Reference: [OpenAI image-input documentation](https://developers.openai.com/api/docs/guides/images-vision).

