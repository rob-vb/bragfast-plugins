---
name: bragfast
description: >
  Generate branded release announcement images and videos using Bragfast.
  Auto-reads git history to compose slides with zero input. Use this skill
  when the user wants to create release images, release videos, visual
  changelogs, announcement slides, branded sprint review visuals, or share
  what they shipped. Also use when they say "make something shareable",
  "show off my work", "visual changelog", or want images/video of their
  recent commits or features. Do NOT use for text-only content like written
  release notes, blog posts, PR descriptions, or markdown changelogs.
---

# /bragfast — Generate Release Content

Create branded release announcement images or videos from the user's recent work.

## Step 1: Gather Content

Try these sources in order — use the first one that yields results:

**Git history (preferred):** If in a git repo, look at recent changes. On a feature branch, diff against the default branch. On main/master, check the last ~5 commits. Extract **3-5 announcement-worthy changes**, prioritizing user-facing features over internal refactors. If the diff is huge (500+ lines), focus on commit messages and user-facing file types (`.tsx`, `.jsx`, `.swift`, `.kt`, `.vue`, `.svelte`, `.html`, `.css`).

**Conversation context:** Scan the conversation for features the user built, bug fixes, version numbers, changelogs, or screenshots/URLs they mentioned.

**Ask the user:** If neither source yields content, ask: "What should the release images cover? You can paste a GitHub release URL, describe the features, or say 'video' for a video."

### Present for approval

Show proposed slides as a numbered list:

```
Here's what I'd put on the slides based on your recent changes:

1. **[Title]** — "[Description]"
2. **[Title]** — "[Description]"
3. **[Title]** — "[Description]"

You can edit, remove, or add slides — or say "looks good" to continue.
```

After the user approves (or adjusts), use `AskUserQuestion` for up to three questions. Skip any question the user already answered in their request:

1. **Output type** (single-select):
   - Question: "What would you like to generate?"
   - Options: "Images" (static slides), "Video" (animated release video), "Both" (images + video)

2. **Formats** (multi-select with `multiSelect: true`):
   - Question: "Which formats do you want?"
   - Options: "Landscape" (Twitter/X, blogs, newsletters), "Portrait" (Instagram Stories, TikTok), "Square" (LinkedIn, Instagram feed)

3. **Screenshots** (single-select):
   - Question: "Do you have screenshots or images to include on the slides?"
   - Options: "Yes, I have local files" (use `bragfast_upload_image` to upload each file, then use the returned URL as `image_url` on the relevant slides), "Yes, I'll paste URLs" (wait for the user to provide image URLs, then map them to the `image` object ID on the relevant slides), "No, text only" (skip image objects)

## Step 2: Brand & Template Setup

1. Call `bragfast_check_account`, `bragfast_list_brands`, and `bragfast_list_templates` in parallel
2. If credits are low, warn the user before proceeding
3. If only one brand exists, use it automatically — don't ask. If multiple, pick the one matching the repo name. If no clear match, ask
4. If no brands exist, ask for colors (background, text, primary as hex)

Pick a template based on context:
- Mobile work (React Native, Swift, Flutter, paths with `ios/`, `android/`) → `*-mobile` template
- Web/dashboard/browser UI → `*-browser` template
- Marketing, launches, or unclear → `hero` template

Present your suggestion with brief reasoning and let the user confirm or change.

## Step 3: Compose Slides

1. Call `bragfast_get_template` for the chosen template to get object IDs
2. For each slide, compose a short punchy title (~40 chars max) and 1-2 line description
3. Map content to object IDs from the template config (`title`, `description`, `image`)
4. Show the slide plan and get approval before generating

## Step 4: Generate

### Gotchas

- The `formats` parameter must be a JSON array of objects, not a JSON string. Pass `"formats": [{"name": "landscape", "slides": [...]}]`, not `"formats": "[{...}]"`.

### For images:
1. Call `bragfast_generate_release_images` with the composed slides
2. Poll `bragfast_get_render_status` with the returned `cook_id` — wait **60 seconds** before the first check, then **30 seconds** between subsequent checks, up to 5 attempts total

### For video:
1. Call `bragfast_generate_release_video` with the composed slides
2. Poll `bragfast_get_render_status` — wait **60 seconds** before the first check, then **30 seconds** between subsequent checks, up to 8 attempts total (videos take longer)

### After results:
- Show the image/video URLs
- Report credits used and remaining
- Offer to generate in other formats or as video/images if they only did one

## Error Handling

- **Not authenticated:** Tell the user to run `npx @bragfast/mcp-server login` or set `BRAGFAST_API_KEY`
- **Insufficient credits:** Show credits needed and link to billing
- **Render fails:** Show the error — credits are auto-refunded
