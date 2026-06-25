# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A single-page marketing website for a fictional investment advisory firm, **Sterling & Vale Advisory**. The entire site is one self-contained file: [index.html](index.html) — HTML, an embedded `<style>` block, and an embedded `<script>` block. There is **no build step, no package manager, no test suite, and no dependencies** except Google Fonts loaded via `<link>`. Everything is vanilla HTML/CSS/JS by design — keep it that way (no frameworks, no bundlers, no external JS/CSS libraries).

## Running / previewing

- Open the file directly in a browser (`file://`), or serve the folder: `python -m http.server` then visit `localhost:8000`.
- There is nothing to build, lint, or test via tooling; "testing" means opening the page and exercising it manually (responsiveness at ~375/768/1280px, scroll animations, the testimonial carousel, and the form's validation/sending/success/error states).

## Architecture

The `<style>` block is organized top-down: design tokens → base/reset → component sections (navbar, hero, services, testimonials, form, footer) → reveal animation → responsive `@media` blocks at the bottom. All colours and spacing come from `:root` custom properties (`--navy`, `--gold`, `--space-*`, etc.) — change tokens there rather than hardcoding values.

The `<script>` block is a series of self-invoking IIFEs, one per concern, each guarded against missing elements: mobile nav toggle, smooth-scroll, IntersectionObserver scroll reveal (adds `.in-view` to `.reveal` elements; respects `prefers-reduced-motion`), the testimonial carousel (auto-rotate + prev/next/dots, active only below 1024px where CSS hides all-but-one slide), footer year injection, and the enquiry form.

Responsive behaviour is CSS-driven: the testimonial track is a static 3-column grid ≥1024px and a JS one-at-a-time carousel below that; the nav collapses to a hamburger ≤860px.

## Enquiry form (the one stateful piece)

Submits via **FormSubmit's AJAX endpoint** using `fetch()` — no page redirect. Key points when editing:

- The destination address is a constant `ENDPOINT` inside the form IIFE, flanked by `REPLACE_WITH_YOUR_EMAIL` comments. It currently posts to `darren.tan@redbeaconam.com`.
- The JSON body must keep the FormSubmit helper fields: `_subject`, `_template: "table"`, `_captcha: "false"`.
- Client-side validation (required name/email/message + email regex) runs before sending; a hidden `_honey` honeypot field silently aborts bot submissions.
- **Activation gotcha:** FormSubmit sends a one-time confirmation email on the first submission to any new address; the form delivers nothing until that link is clicked.
