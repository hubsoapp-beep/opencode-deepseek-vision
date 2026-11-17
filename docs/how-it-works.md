# How it works

Detailed walkthrough of the `opencode-vision-router` plugin used by this recipe.

## Request flow

1. User pastes an image into the OpenCode chat.
2. The plugin's `chat.message` / `session.hook("context")` hook fires before the main model sees
   the message.
3. The image part (data URL, `file://`, absolute path, or V2 media part) is resolved to a readable
   file path and written to the temp directory (content-hashed, reused across calls).
4. The image part is replaced with a text pointer:

   ```
   [The user attached an image, saved at: /tmp/opencode-vision/<hash>.png]
   Use the "vision" subagent (via the Task tool) to analyze this image and answer the user's
   request about it. Pass the path and the request to the subagent.
   ```

5. The main (text-only) model receives the pointer and invokes the `vision` subagent via the Task tool.
6. The `vision` subagent reads the image file and returns a textual analysis.
7. The main model uses that analysis to answer the original question.

## Capability detection

- If the main model is multimodal and `force` is `false`, routing is skipped and the image is
  passed through untouched.
- With `force: true`, routing always happens (useful to send images to a cheaper vision model).

## Tool-result safety

Images embedded in tool results (for example, the `read` tool reading an image file) are stripped
the same way, so they are not replayed to a text-only model on later steps.

## Permissions of the vision subagent

- `external_directory`: allow (read the image file)
- `bash`: deny
- `edit`: deny
- `webfetch`: deny
- `doom_loop`: deny
