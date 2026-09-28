# OpenJEV Support

This fork adds optional [OpenJEV](https://openjev.sh) support alongside the original
[TypeSafe](https://typesafe.ai) integration. OpenJEV is a free community gateway to the
same Jev model built by TypeSafe. TypeSafe remains the default; anyone with a TypeSafe
key sees zero behaviour change.

## What was added

- **`src/kedi_typesafe/core/evaluation.py`** — Provider resolution logic
  (`_resolve_provider`) and `base_url` passed to `TypeSafeClient`/`AsyncTypeSafeClient`
  constructors. New `provider` parameter on `TypeSafeEvaluator`; `base_url` property.
- **`src/kedi_typesafe/integrations/_pydantic_provider.py`** — `name` and `base_url`
  properties now reflect the active provider instead of hardcoding TypeSafe.
- **`src/kedi_typesafe/integrations/pydantic.py`** — `provider` parameter forwarded to
  `TypeSafeEvaluator`.
- **`src/kedi_typesafe/integrations/langchain.py`** — `provider` parameter forwarded to
  both `TypeSafeEvaluator` and `ClassifierTransport`.
- **`src/kedi_typesafe/integrations/_langchain_transport.py`** — `base_url` passed to
  `TypeSafeClassifier`; resolved key and default model follow the provider selection.
- **`README.md`** — Short note after the project intro.

No TypeSafe code was renamed, removed, or re-defaulted.

## Provider selection rule

1. **Explicit choice wins** — pass `provider="openjev"` (or `provider="typesafe"`) to any
   constructor, or set `JEV_PROVIDER=openjev` in the environment.
2. **Otherwise, if `TYPESAFE_API_KEY` is set** → TypeSafe (default, unchanged).
3. **Otherwise, if only `OPENJEV_API_KEY` is set** → OpenJEV.

When OpenJEV is selected and the model name is still the default `jev-latest`, it is
automatically changed to `openjev` (the model id expected by the OpenJEV endpoint).

## Configuration

```bash
# Option A: auto-detect (only OpenJEV key set)
export OPENJEV_API_KEY="your-openjev-key"

# Option B: explicit provider
export TYPESAFE_API_KEY="your-typesafe-key"
export OPENJEV_API_KEY="your-openjev-key"
export JEV_PROVIDER=openjev
```

```python
# Option C: per-instance
from kedi_typesafe import TypeSafeEvaluator
from kedi_typesafe.integrations.pydantic import TypeSafeModel

model = TypeSafeModel(provider="openjev")
```

## Verification

A live `POST https://api.openjev.sh/v1/systemone` request with model `openjev`, state
`ping`, and one `noul` question returned HTTP 200. No repository code was executed.

## Upstream

Original project: https://github.com/kedi-lang/kedi-typesafe by @kedi-lang.
