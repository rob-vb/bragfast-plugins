---
name: bragfast
description: Generate branded release announcement images and videos. Auto-reads your git history to compose slides with zero input.
---

# /bragfast — Generate Release Content

You are helping the user create branded release announcement images or videos using Bragfast.

## Step 0: Read the Code

Before reading conversation context, check if you're in a git repository with recent changes. This lets you auto-generate slide content from what the user just built.

1. Check if in a git repo: run `git rev-parse --is-inside-work-tree 2>/dev/null`. If this fails, skip to Step 1.
2. Detect the current branch: `git branch --show-current 2>/dev/null`
   - If on a **feature branch** (anything other than `main` or `master`): detect the default branch via `git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's|refs/remotes/origin/||'` (falls back to `main` if this fails). Then run `git diff <default-branch> --stat` to see all changes on this branch.
   - If on **main/master**: run `git diff HEAD~5 --stat` to see recent changes.
3. Get recent commit messages: `git log --oneline -5`
4. **Large diff handling:** If the diff stat shows more than 500 lines changed (insertions + deletions), filter to user-facing files only: `git diff <target> -- '*.tsx' '*.jsx' '*.swift' '*.kt' '*.vue' '*.svelte' '*.html' '*.css'`. If still over 500 lines, use only the commit messages from step 3 and the file stat summary.
5. From the git output, extract **3-5 announcement-worthy changes**. Prioritize:
   - User-facing features over internal refactors
   - New capabilities over bug fixes (unless the fix is notable)
   - Changes with descriptive commit messages
6. Note the file types changed — this informs template selection in Step 3.

**If not in a git repo, no commits found, or git commands fail:** Skip to Step 1.
**If the diff is empty or only contains chore/config changes:** Skip to Step 1.
**If in a shallow clone** (git log shows only 1 commit and diff fails): Use `git log -1` message only.
**If in detached HEAD state** (`git symbolic-ref --short HEAD` fails): Use `git log -1` message only.

**If Step 0 produced slide content:** Present it using the approval gate format below, then proceed to Step 2 for brand/template selection.

### Approval gate

Present proposed slides as a numbered list:

```
Here's what I'd put on the slides based on your recent changes:

1. **[Title]** — "[Description]"
2. **[Title]** — "[Description]"
3. **[Title]** — "[Description]"

Template: [auto-suggested based on file types]
Brand: [auto-selected or only brand]

**Output type** (pick one):
- [ ] Images — static slides
- [ ] Video — animated release video
- [ ] Both

**Formats** (pick one or more):
- [ ] Landscape — best for Twitter/X, blogs, newsletters
- [ ] Portrait — best for Instagram Stories, TikTok
- [ ] Square — best for LinkedIn, Instagram feed
- [ ] All three

Just tell me what you want, e.g. "images in landscape and square" or "both, all formats".

You can also edit slides, remove/add slides, or change the template.
```

The user responds in natural language. Interpret their response and either adjust or proceed to generation.

---

## Step 1: Read the Room

**If Step 0 already produced slide content from git context, skip to Step 2.**

Otherwise, scan the conversation history for context. Look for:
- Features the user just built or shipped
- Bug fixes, refactors, or improvements discussed
- Version numbers, release notes, or changelogs mentioned
- Screenshots or URLs that could be used as slide images
- Any description of what changed and why

**If you find relevant context:** Propose slide content based on what you found. Say something like: "Based on what you've been working on, here's what I'd put on the slides:" — then show the plan. The user can approve or adjust.

**If the conversation is empty or unrelated:** Ask: "What should the release images cover? You can paste a GitHub release URL, describe the features, or say 'video' for a video."

## Step 2: Gather Brands & Templates

1. Call `bragfast_list_brands` and `bragfast_list_templates` in parallel
2. If only one brand exists, use it automatically — don't ask
3. If multiple brands, select the brand whose name most closely matches the current repo name. If no clear match, ask the user which one
4. If no brands, ask for colors (background, text, primary as hex)

## Step 3: Pick Template & Format

Based on the conversation context and file types from Step 0 (if available), suggest a template and format that fits:

**If Step 0 detected file types**, use this priority matrix:

| Files changed | Template | Format |
|---|---|---|
| `.swift`, `.kt`, or paths containing `ios/`, `android/`, `react-native`, `expo` | `*-mobile` | portrait |
| `.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.css` | `*-browser` | landscape |
| Mixed mobile + web (mobile-specific paths present) | `*-mobile` | portrait |
| Mixed mobile + web (no mobile-specific paths) | `*-browser` | landscape |
| Backend-only, API, or unclear | `hero` | landscape |

**Otherwise**, use conversation context:

- **Mobile app work** (React Native, Swift, Flutter, responsive design) → suggest a `*-mobile` template + portrait or square format
- **Web app / dashboard / browser UI** → suggest a `*-browser` template + landscape format
- **Hero/marketing content** (launch, announcement, blog) → suggest `hero` template + landscape format
- **Social media** → suggest square format
- If the user has custom templates, check if any match the context by name

Present your suggestion with reasoning: "Since you've been building a mobile feature, I'd suggest **standard-mobile** in **portrait** — but you can pick a different one." Let the user confirm or change.

## Step 4: Compose Slides

From the conversation context, release notes, or user description:

1. Extract the top 3-5 features or changes
2. Call `bragfast_get_template` for the chosen template to get object IDs
3. For each slide, compose:
   - A short, punchy title (max ~40 chars)
   - A 1-2 line description
   - Map content to object IDs from the template config (`title`, `description`, `image`)
4. Show the slide plan and ask for approval

## Step 5: Generate

**CRITICAL: The `formats` parameter must be a JSON array, not a string.** Pass it as:

```json
{
  "formats": [
    {
      "name": "landscape",
      "slides": [
        {
          "objects": [
            { "id": "title", "text": "My Title" },
            { "id": "description", "text": "My description" }
          ]
        }
      ]
    }
  ]
}
```

Do NOT pass `formats` as a JSON string like `"[{...}]"` — it must be an actual array of objects.

### For Images:
1. Call `bragfast_generate_release_images` with the composed slides
2. You'll get back a `cook_id`
3. Wait 5 seconds, then call `bragfast_get_render_status` with the cook_id
4. If still pending, wait another 5 seconds and check again (max 3 attempts)
5. When complete, show the image URLs to the user

### For Video:
1. Call `bragfast_generate_release_video` with the composed slides
2. You'll get back a `cook_id`
3. Wait 10 seconds, then call `bragfast_get_render_status`
4. Video renders take longer — check up to 6 times at 10-second intervals
5. When complete, show the video URL

## Step 6: Follow Up

After showing results:
- Show credits used and remaining
- Ask: "Want me to generate this in other formats? Or create a video version?"

## Error Handling

- If not authenticated: Tell the user to run `npx @bragfast/mcp-server login` or set `BRAGFAST_API_KEY`
- If insufficient credits: Show the credits needed and link to billing
- If render fails: Show the error and mention credits are auto-refunded

## Tips for Good Results

- Keep slide titles short and impactful
- Use "browser" or "mobile" device frames for screenshots
- Square format works best for social media, landscape for blogs/newsletters
- If the user has been discussing a UI change, suggest they provide a screenshot URL for the image object
