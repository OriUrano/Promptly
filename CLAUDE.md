# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About Promptly

Promptly is a monologue memorization app built with SvelteKit 5. It helps users practice memorizing text by breaking it into lines and using an interactive word bank system where users reconstruct each line from scrambled words mixed with decoys.

## Development Commands

- `npm run dev` - Start development server (avoid running this automatically)
- `npm run build` - Create production build
- `npm run preview` - Preview production build
- `npm run check` - Run Svelte type checking
- `npm run check:watch` - Run Svelte type checking in watch mode
- `npm run format` - Format code with Prettier
- `npm run lint` - Check code formatting with Prettier

## Architecture Overview

This is a single-page application with the main functionality contained in `src/routes/+page.svelte`. The app operates in three main modes:

### Application State Management
- Uses Svelte 5's `$state()` runes for reactive state management
- State includes monologue text, practice mode, current line tracking, word arrays, and user input
- LocalStorage integration for persisting monologue data

### Core Application Modes
1. **Input Mode**: User enters monologue text in a textarea
2. **Practice Mode**: Interactive word selection interface for reconstructing lines
3. **Completed Mode**: Celebration screen with options to restart

### Word Bank Logic
- `generateScrambledWords()`: Creates word bank by combining current line words with decoy words from other lines
- `addWordToSubmission()`: Moves words from available pool to user selection (handles duplicates by index)
- `removeWordFromSubmission()`: Returns words from selection back to available pool
- Word deduplication and randomization using Fisher-Yates shuffle

### Key Technical Details
- Uses `@sveltejs/adapter-static` for static site generation
- Tailwind CSS v4 for styling with custom animations
- TypeScript support via JSDoc comments
- Word processing includes punctuation removal and case normalization
- Previous line context display with "First Line" fallback for first line

## File Structure
- `src/routes/+page.svelte` - Main application component containing all logic
- `src/routes/+layout.svelte` - Basic layout wrapper
- `src/app.html` - HTML template
- `src/app.css` - Global styles (minimal, mostly Tailwind)

## Important Implementation Notes
- Duplicate word handling in word bank uses index-based removal to prevent removing all instances
- Line progression automatically advances on correct answers with 1.5s delay
- Feedback system shows correct/incorrect state with retry and skip options
- Progress tracking via visual progress bar and line counters