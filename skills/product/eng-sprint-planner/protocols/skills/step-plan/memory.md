# Step Plan — Memory

## Investigation spikes

- **Kernel v3 internals**: When a macro step touches Kernel v3 signature handling (`_checkSignaturePolicy`, `isValidSignature`, signature advancement, policy data), propose an investigation spike before committing to an implementation approach. Kernel v3 reads `sig[0]` on the original unadvanced signature, which limits what can be injected via the signature payload. Sprint 0002 required 3 iterations on Path B signature design because this behavior wasn't understood upfront.
