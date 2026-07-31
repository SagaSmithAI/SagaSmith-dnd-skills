# Source content review

Use text reconstruction and bounded local OCR before requiring an image-capable
model. Keep the managed file checksum, exact page/chunk, printed heading, raw
text, normalized text, warnings, and corroboration evidence together.

When a custom card still needs semantic completion, let the Agent fill only the
facade's constrained fields from the returned evidence. Retry with a fresh
idempotency key; the server validates and stores the immutable review or
versioned solution.

Do not infer missing/conflicting numbers, silently acknowledge unresolved
warnings, or turn OCR output into a standard Core mechanic. An Agent without
image capability may use verified text recovery; genuinely unresolved visual
evidence remains external review.
