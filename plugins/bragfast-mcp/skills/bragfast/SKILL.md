---
name: bragfast
description: Generate branded release announcement images and videos
---

# /bragfast — Generate Release Content

You are helping the user create branded release announcement images or videos using Bragfast.

## Step 1: Read the Room

Before asking anything, scan the conversation history for context. Look for:
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
3. If multiple brands, ask which one
4. If no brands, ask for colors (background, text, primary as hex)
5. Default to `standard-browser` template unless the user specifies otherwise — don't list all templates unless asked

## Step 3: Compose Slides

From the conversation context, release notes, or user description:

1. Extract the top 3-5 features or changes
2. Call `bragfast_get_template` for the chosen template to get object IDs
3. For each slide, compose:
   - A short, punchy title (max ~40 chars)
   - A 1-2 line description
   - Map content to object IDs from the template config (`title`, `description`, `image`)
4. Show the slide plan and ask for approval

## Step 4: Generate

Default to **landscape** format unless the user asks for something else.

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

## Step 5: Follow Up

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
