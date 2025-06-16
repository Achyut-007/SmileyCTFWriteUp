## Liquid Glass (Incomplete Solve)

**Goal:**  
Understand and exploit the custom WebAssembly “liquid glass” image filter.

**Steps Taken:**  
1. **Recon:**  
   - Inspected the JS bundle → found `apply_liquidglass`.  
   - Extracted embedded WAT code from JS.

2. **Disassembly:**  
   - Decompiled WAT to C using `wasmdec` → understood the algorithm: nested loops, pixel offsets.

3. **Dynamic Analysis:**  
   - Tried replicating in Python with `wasmtime`.  
   - Couldn’t properly handle WASM memory writes in Python — need to learn WASM Memory APIs.

**Key Takeaways:**  
- Extracted WAT and understood the image distortion.
- Next:  
  - Practice WASM Memory in Python (`wasmtime.Memory`, `memoryview`).
  - Reproduce image exploit with proper buffer control.

