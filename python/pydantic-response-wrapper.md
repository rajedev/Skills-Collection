# Skill: Pydantic Response Wrapper

---
name: pydantic-response-wrapper
description: Use this skill when building any models/ file. Defines standard response wrapper, input validator pattern, nullable field rule, and file upload validation pattern.
---

## Standard Wrapper
Every assignment wraps its output in a response envelope:

class <Domain>Response(BaseModel):
    success: bool
    data: <OutputModel> | None = None
    error: str | None = None

## Input Validator Pattern
Use field_validator for input models:

class <Domain>Request(BaseModel):
    <field>: str

    @field_validator("<field>")
    @classmethod
    def must_not_be_empty(cls, v):
        if not v or not v.strip():
            raise ValueError("<field> must not be empty")
        return v.strip()

## Nullable Field Rule
If a field should be null when not found in source text:
    field: str | None = None

Always default to None, never guess or infer.
Document in a comment why it is nullable.

## File Upload Validation Pattern
For UploadFile endpoints:

async def upload(file: UploadFile) -> <Domain>Response:
    if not file.filename.endswith(".txt"):
        raise HTTPException(status_code=422,
            detail="Only .txt files accepted")
    content = await file.read()
    transcript = content.decode("utf-8")
    request = <Domain>Request(<field>=transcript)
    return await service.process(request)
