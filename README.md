# CoachGPT
CoachGPT is a personal fitness coaching system that connects Apple Health data to ChatGPT, allowing you to ask natural-language questions about your training, recovery, and progress.

It is designed to act as a holistic coach, combining:
	•	your personal workout and health data
	•	general training and physiology knowledge
	•	contextual reasoning over time (trends, baselines, fatigue)

CoachGPT currently runs entirely through the ChatGPT UI, with a private backend used only to store and retrieve health data.

# What CoachGPT Can Do
Examples of questions CoachGPT is designed to answer:
> 
	•	“How’s my training today compared to my recent baseline?”
	•	“Why did my running performance drop this week?”
	•	“How did yesterday’s cycling session go?”
	•	“Was my heart rate drifting on my last run?”
	•	“Should I continue with my current training plan?”

CoachGPT determines what data to fetch automatically and explains results in plain language.

# High-level Architecture

Apple Health

↓

Health Auto Export (iOS)

↓

Supabase Edge Function (ingest)

↓

Postgres (workouts + samples)

↓

Supabase Edge Function (read-only API)

↓

ChatGPT Custom GPT (Actions)

Key Design Principles
	•	ChatGPT is the front-end
	•	No custom app or UI
	•	All interaction happens in the ChatGPT interface
	•	Server-side data access
	•	ChatGPT never accesses the database directly
	•	All reads go through controlled API endpoints
	•	Separation of concerns
	•	Workout summaries and time-series samples are stored separately
	•	Large data (GPS, HR buckets) is only fetched when needed


# Data Model Overview
workouts

Stores one row per workout (run, cycle, etc.).

Examples of fields:
	•	activity type (run, cycling, etc.)
	•	duration
	•	distance
	•	active energy
	•	average / min / max heart rate
	•	elevation gain
	•	environment (indoor/outdoor)

workout_samples

Stores time-series data associated with a workout.

Examples:
	•	heart rate buckets
	•	step count per minute
	•	distance buckets
	•	active energy over time

This separation keeps summary queries fast and sample queries flexible.

# API (Read-Only)
CoachGPT exposes a read-only API via a Supabase Edge Function.
ChatGPT accesses this API using Actions defined via an OpenAPI schema.

Available Endpoints

1. Today Summary
GET /summary/today
Returns:
	•	today’s workouts
	•	a baseline window (default: last 28 days)
	•	workouts in that baseline window

Used for questions like:

“How’s my training today?”

2. Workouts by Date Range
GET /workouts?from=&to=&limit=
Returns:
	•	workouts between two timestamps

Used to:
	•	resolve natural-language references (e.g. “yesterday’s run”)
	•	compare weeks or months
	•	find a specific workout before deeper analysis

3. Workout Samples
GET /workouts/{external_id}/samples
Returns:
	•	time-series data (HR, distance, steps, etc.) for a specific workout

Used for:
	•	HR drift analysis
	•	interval inspection
	•	detailed performance breakdowns

Note: Users never provide external_ids directly.
ChatGPT infers the correct workout by calling the range endpoint first.

# Authentication

All API access is protected via a shared secret passed in a custom HTTP header.
	•	No public access
	•	No user accounts
	•	No OAuth (for now)

This keeps the system simple and private.

# Privacy & Data Handling
	•	All data originates from Apple Health via explicit user export
	•	Data is stored in a private Supabase database
	•	Data is never sold or shared
	•	No analytics, ads, or tracking

See PRIVACY.md￼ for full details.

# Why This Exists

Most fitness apps:
	•	show dashboards
	•	surface metrics
	•	offer no interpretation, explanation, or reasoning

CoachGPT is different:
	•	you ask questions
	•	it finds the relevant data
	•	it explains why things are happening
	•	it adapts as your training changes

This project is about context, reasoning, and insight, not charts.
