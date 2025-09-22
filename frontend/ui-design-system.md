# UI Design System

The UI uses Bootstrap 5 as its foundation with custom CSS for application-specific components. The design system covers theming, zoom scaling, responsive layout, and component styling.

---

## What You Will Learn

- How theming works (light mode, dark mode)
- How zoom scaling affects the layout
- How responsive design adapts to different screen sizes
- How CSS variables are used for consistency

---

## Theme System

The application supports multiple color themes. The theme context (`ThemeContext.js`) manages the active theme and provides it to all components:

The theme context provides the current theme value and a setter that persists the preference to localStorage.

Themes are defined through CSS variables in `theme-colors.css`. Each theme variant overrides color variables:

Themes are defined through CSS variables on the `:root` and `[data-theme]` selectors. Switching themes changes the data attribute on the root element, and all colors update automatically without re-renders.

Theme preference is persisted in localStorage under `colorTheme` and `colorBasedTheme` keys.

---

## Zoom Scaling

The application supports zoom levels from 75% to 100%. The ScaleWrapper component applies a CSS transform to scale the entire UI:

The ScaleWrapper component applies CSS `transform: scale()` to the entire UI based on the user's preferred zoom level. Zoom levels range from 75% to 100% in 5% increments.

The zoom level affects font sizes, spacing, chart dimensions, and table layouts. The system stores the preferred zoom level in localStorage under `uiScale`.

---

## Chart.js Zoom Integration

Chart.js tooltips break at zoom levels below 100% because the library's hit detection uses uncorrected mouse coordinates. A custom `beforeEvent` plugin divides coordinates by the zoom factor:

A custom `beforeEvent` Chart.js plugin divides mouse coordinates by the zoom factor before Chart.js processes them for hit detection. This fixes tooltip positioning at every zoom level.

This fix applies to all charts, ensuring tooltips and hover states work correctly at any zoom level.

---

## Responsive Layout

The layout uses Bootstrap's grid system (`.row`, `.col-*`, `.col-md-*`, `.col-sm-*`) for responsive behavior. Key breakpoints:

| Breakpoint | Max Width | Layout                               |
| ---------- | --------- | ------------------------------------ |
| xs         | <576px    | Single column, stacked navigation    |
| sm         | 576px     | Two columns, compact sidebar         |
| md         | 768px     | Full sidebar, multi-column layout    |
| lg         | 992px     | Full dashboard layout                |
| xl         | 1200px    | Maximum width, spacious layout       |

---

## Custom CSS Architecture

Custom styles are organized by component and feature:

- `style-v8.css` - Global application styles
- `responsive-v6.css` - Responsive overrides
- `theme-colors.css` - Theme variable definitions
- Component-specific CSS where needed

The custom table, date picker, and notification panel have dedicated CSS files alongside their components.

---

## Interview Talking Points

**On the zoom correction plugin:** "Chart.js uses getBoundingClientRect to map mouse coordinates, but at 75% zoom the coordinates are scaled. I wrote a beforeEvent plugin that divides coordinates by the zoom factor before Chart.js processes them for hit detection. It is a small fix but without it, tooltips would be offset at any non-default zoom level."

**On the theme system:** "Themes are defined through CSS variables. Switching themes just changes the data attribute on the root element, and all colors update automatically. This is simpler than context-based theming and avoids re-renders when the theme changes."

## Related Documents

- [Frontend Architecture](../architecture/frontend-architecture.md)
- [Custom Table System](custom-table-system.md)
- [Component Architecture](component-architecture.md)
