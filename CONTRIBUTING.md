# Contributing to BULB

Thanks for checking out BULB! 👋

I'm happy that you're interested in contributing. Whether it's fixing a bug, improving performance, enhancing the visuals, or suggesting a new feature, every contribution is appreciated.

Before opening an issue or pull request, please read the guidelines below.

---

## Code of Conduct

Please be respectful and constructive when interacting with others. This project follows the Contributor Covenant Code of Conduct.

https://www.contributor-covenant.org/version/2/0/code_of_conduct/

---

# Reporting Bugs

Before opening a new issue, please check if someone has already reported the same problem.

When reporting a bug, try to include:

- Browser and version
- Operating system
- Device (Desktop/Mobile)
- Steps to reproduce the issue
- Expected result
- Actual result
- Screenshots or recordings if possible

### Example

```text
Title: Audio reactivity not working on Firefox

Environment:
Firefox 120
Windows 11
AMD GPU

Steps:
1. Click Mic
2. Allow microphone access
3. Play music

Expected:
The fractal reacts to audio.

Actual:
No audio response even though the microphone is active.
```

---

# Suggesting Features

Ideas are always welcome.

If you're requesting a feature, try to explain:

- What the feature does
- Why it would improve BULB
- How you imagine it working

Mockups, sketches, or reference images are always helpful but not required.

---

# Development Setup

### 1. Fork the repository

Create your own fork on GitHub.

### 2. Clone your fork

```bash
git clone https://github.com/YOUR-USERNAME/bulb.git
cd bulb
```

### 3. Create a branch

```bash
git checkout -b feature/my-feature
```

### 4. Make your changes

Please test your changes before opening a pull request.

If your work affects rendering or shaders, try testing in at least Chrome and Firefox.

### 5. Commit

This project uses Conventional Commits.

Examples:

```bash
git commit -m "feat: add bloom intensity control"
git commit -m "fix: correct camera reset logic"
git commit -m "docs: improve README examples"
```

### 6. Push

```bash
git push origin feature/my-feature
```

### 7. Open a Pull Request

Describe what you changed and why.

If your PR fixes an issue, mention it using:

```
Fixes #12
```

---

# Pull Request Checklist

Before opening a PR, please make sure:

- [ ] The project builds successfully
- [ ] No unnecessary files are included
- [ ] The code follows the existing style
- [ ] Documentation is updated if needed
- [ ] No new console errors or warnings appear
- [ ] Changes have been tested

---

# Code Style

The project doesn't use a strict formatter, but please try to follow the existing style.

### General

- Keep code easy to read.
- Prefer meaningful variable names.
- Keep functions focused on one task.
- Add comments only when they explain **why**, not **what**.

### JavaScript

```javascript
function calculateDistance(a, b) {
  return Math.abs(a - b);
}

const maxIterations = 14;
let currentFrame = 0;
```

Use:

- `const` whenever possible
- `let` when values change
- Avoid `var`

---

### CSS

Use CSS variables whenever possible.

```css
:root {
  --color-primary: #c9903f;
  --color-text: #ece6da;
}

.panel {
  background: var(--panel);
  border: 1px solid var(--line);
}
```

Try to avoid deeply nested selectors.

---

### GLSL

Keep shader code readable.

Use descriptive variable names and add comments around complex calculations.

```glsl
// Distance estimator
float mapDE(vec3 p) {
    ...
}
```

---

# Performance

BULB is GPU-heavy, so performance matters.

When possible:

- Avoid unnecessary calculations in shaders.
- Don't allocate new objects every frame.
- Cache DOM lookups.
- Reuse buffers instead of recreating them.
- Clean up textures and WebGL resources when they're no longer needed.

---

# Testing

Before opening a PR, check that everything still works.

Recommended browsers:

- Chrome
- Firefox
- Safari (if available)

Also verify:

- Desktop layout
- Mobile layout
- No console errors
- Smooth rendering performance
- Audio reactivity (if modified)

---

# Documentation

If your changes affect how BULB works, please update the documentation.

That may include:

- README.md
- Examples
- Comments for complex code

---

# Commit Messages

Use Conventional Commits.

Examples:

```text
feat: add bloom effect

fix: resolve WebGL context loss

docs: update installation guide

perf: reduce shader iterations

refactor: simplify render pipeline
```

Common prefixes:

- feat
- fix
- docs
- style
- refactor
- perf
- test
- chore

---

# Need Help?

If you have questions, feel free to:

- Open a GitHub Discussion
- Open an Issue
- Contact me at **gagananair1@gmail.com**

---
