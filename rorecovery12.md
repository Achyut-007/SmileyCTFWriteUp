## Rorecovery12 (Incomplete Solve)

**Challenge:**  
- Single unknown file, no description or URL.  
- Goal: Find a hidden flag or secret related to Roblox.

**Steps Taken:**  
1. **Identified File:**  
   - Ran `file` → no useful output.  
   - Opened in hex viewer — saw `<rblx!` → confirmed `.rbxm` Roblox binary model.

2. **Tried Opening:**  
   - Roblox Studio failed to load it.

3. **Found Asset IDs:**  
   - Searched hex dump for `SourceAssetId` and other numbers.
   - Downloaded each asset manually.

4. **Automated with Bash:**  
   ```bash
   #!/usr/bin/env bash

   # Extract IDs from hex dump
   grep -oE '[0-9]{6,}' hexdump.txt | sort -u > ids.txt

   # Download each ID
   mkdir -p assets
   while read id; do
     echo "Trying ID: $id"
     curl -s -o "assets/$id" "https://assetdelivery.roblox.com/v1/asset?id=$id"
   done < ids.txt

   echo "Done."
   ```

5. **What I Found:**  
   - Some assets were ZIPs — extracted images, symbols, etc.
   - Found a `flip` animation name, but no SourceAssetID associated with it.

**Lessons Learned:**  
- Analyzed Roblox binary models with `xxd` and `hexdump`.
- Scraped numeric asset IDs.
- Automated brute-force asset recovery with Bash.

**Current Status:**  
- Extracted all visible IDs.
- Couldn’t find flag.

