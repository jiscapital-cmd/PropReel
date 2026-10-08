# `media`

Handle phone video: signed upload to S3/R2, FFmpeg preview, transcription (Groq Whisper API), auto-tags, transcript search (pgvector).

**PRD:** §12 Media  
**Owns tables:** media fields on `content_piece`  
**Acceptance criteria:** AC-11

Target: a 2-minute video is transcribed and searchable within 5 minutes.

**Must never**
- Auto-edit or re-cut video (V1 non-goal)

*Placeholder: no code yet.*
