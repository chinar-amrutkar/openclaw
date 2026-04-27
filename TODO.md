# Implementation Plan for PR #71459 Review Comments

## Problem
PR #71459 removes `formatMissingPluginRegisterError()` entirely, causing a regression where normal plugins lose debug module shape information when `OPENCLAW_PLUGIN_LOAD_DEBUG=1` is enabled.

## Solution
Restore `formatMissingPluginRegisterError()` and use it selectively:
- For bundled-channel-entry: use special bundled-channel message
- For normal plugins: use `formatMissingPluginRegisterError()` to preserve debug behavior

## Files to Modify
1. `src/plugins/loader.ts` - main plugin loader
2. `src/plugins/loader.test.ts` - add regression tests
3. Possibly `src/channels/plugins/bundled.ts` if needed

## Tasks

### [ ] 1. Restore `formatMissingPluginRegisterError` function in src/plugins/loader.ts
   - Add back the function (it uses existing helpers: `isPluginLoadDebugEnabled`, `describePluginModuleExportShape`)
   - Place it near other helper functions

### [ ] 2. Update plugin loading logic in `loadOpenClawPlugins` function
   - Keep bundled-channel-entry detection using `resolvedModule.kind === "bundled-channel-entry"`
   - For bundled-channel-entry: keep special message
   - For normal plugins: use `formatMissingPluginRegisterError(mod, env)`
   - Keep the `logger.error(...)` call

### [ ] 3. Update CLI plugin loader in `loadOpenClawPluginCliRegistry` function
   - Same pattern as above
   - For bundled-channel-entry: use CLI-specific message
   - For normal plugins: use `formatMissingPluginRegisterError(mod, env)`
   - Keep the `logger.error(...)` call

### [ ] 4. Add regression tests in src/plugins/loader.test.ts
   - Test bundled-channel-entry special message appears
   - Test normal plugin missing register WITH debug enabled includes "module shape:"
   - Test normal plugin missing register WITHOUT debug enabled is plain message
   - Test that bundled-channel-entry detection works correctly (kind field check)

### [ ] 5. Verify no ESLint/lint issues
   - Run `npm run lint` or equivalent
   - Check for curly braces, no-unused-vars, etc.

## Expected Behavior After Fix
- Normal plugins: 
  - Normal mode: `"plugin export missing register/activate"`
  - Debug mode: `"plugin export missing register/activate (module shape: ...)"`
- Bundled-channel-entry plugins: 
  - `"bundled channel entry loaded via legacy plugin loader — use setup-runtime loader instead"`
  - CLI variant: `"bundled channel entry requires setup-runtime loader"`

## References
- Main branch function: `formatMissingPluginRegisterError` 
- Test for debug behavior: `src/plugins/loader.test.ts:4872` ("can include plugin export shape when register is missing")
- Bundled-channel-entry contract: `src/plugin-sdk/channel-entry-contract.ts`