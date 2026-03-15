<overview>
Voice definition, tone modulation, word lists, and the tone-as-dial metaphor. Read when establishing a product voice or adapting tone for a specific interface situation. Related: patterns.md (applying voice to specific UI patterns).
</overview>

## Voice vs Tone

**Voice** is the product's consistent personality — 3-4 qualities that never change.
**Tone** adapts the voice to the situation — dial qualities up or down based on context.

Example voice: "Clear, Warm, Confident"
- Error message: dial warmth UP, confidence DOWN → empathetic, not dismissive
- Success message: dial confidence UP, warmth MEDIUM → celebratory, not cold
- Onboarding: dial warmth UP, clarity UP → welcoming, not overwhelming

## Defining a Voice

### Step 1: Product Identity

Answer these:
- What is the product's core purpose?
- Who is the primary audience?
- In what context do they use it? (stressed? relaxed? professional? casual?)
- If the product were a person, how would it speak?

### Step 2: Choose 3-4 Qualities

Pick qualities that are specific and actionable:

| Good (specific) | Bad (vague) |
|-----------------|-------------|
| Reassuring | Nice |
| Direct | Good |
| Playful | Fun |
| Expert but approachable | Professional |

### Step 3: Define Each Quality

For each quality, write:
- What it means in practice
- What it does NOT mean (the boundary)
- An example sentence that demonstrates it

Example:
> **Direct** — We get to the point. We don't pad with filler or hedge with qualifiers.
> Direct does NOT mean blunt or rude. We're direct AND warm.
> "Your photo was saved." not "We've gone ahead and saved your photo for you!"

## The Tone Dial

Each quality has a dial from 1 (subtle) to 5 (full):

| Situation | Warm | Direct | Playful |
|-----------|------|--------|---------|
| Error (data loss) | 5 | 4 | 1 |
| Success (task done) | 3 | 5 | 3 |
| Onboarding | 5 | 3 | 4 |
| Settings | 2 | 5 | 1 |
| Empty state | 4 | 3 | 4 |

## Word Lists

Maintain consistent vocabulary:

| Use | Don't Use |
|-----|-----------|
| Remove | Delete (unless destructive) |
| Save | Submit (too formal) |
| Done | OK (too ambiguous) |
| Sign in | Log in (Apple style) |
| Turn on | Enable (too technical) |

Project-specific word lists go in the product's voice guide (CLAUDE.md or design system docs).

## Writing Craft Rules

Applied AFTER voice and tone are set:

1. **Remove filler** — cut adverbs, interjections, pleasantries unless they serve the voice
2. **Avoid repetition** — collapse overlapping ideas
3. **Be specific** — name the thing, name the action, give real information
4. **Keep consistent** — same term for same concept everywhere
5. **Use possessives sparingly** — "the account" not "your account" unless personal
6. **Sweat details** — spelling, grammar, capitalization consistency
7. **Build patterns** — familiar moments get familiar language
