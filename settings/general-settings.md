# General Settings

The General Settings tab manages user profile information, security preferences, and display customization. It is the default landing page when users navigate to `/settings/`.

---

## What You Will Learn

- How profile information is displayed and edited
- How password changes work
- How two-factor authentication (2FA) is toggled
- How theme and color themes are managed
- How UI zoom scaling is configured

---

## Profile Information

The General Settings page displays the current user's profile fields from Redux (`state.auth.auth_user`):

| Field              | Editable | Component              |
| ------------------ | -------- | ---------------------- |
| Full Name          | Yes      | EditFullNameModal      |
| Email              | No       | Displayed as read-only |
| Password           | Yes      | ChangePasswordModal    |
| Two-Factor Auth    | Yes      | Toggle switch          |

### Edit Full Name

Clicking "Edit" next to the full name opens a Bootstrap modal (`EditFullNameModal`). The user updates their name and submits. The backend is called via `PATCH /api/v1/user`, and on success, the Redux store and localStorage are updated.

### Change Password

Clicking "Change" next to the password field opens the `ChangePasswordModal`. The user enters their current password and a new password. The backend validates the current password and updates to the new one.

---

## Two-Factor Authentication (2FA)

Users can enable or disable two-factor authentication via a toggle switch. When enabled, login requires an OTP sent via email in addition to the password.

The 2FA toggle uses an optimistic update pattern with promise-based notification feedback. The UI switches immediately while the API call is in flight. If the call fails, the toggle reverts and an error toast is shown.

The toggle uses optimistic UI with `notifyPromise` for feedback. The backend response updates both the Redux store and localStorage user data.

---

## Theme System

The theme section provides three display options:

### Light/Dark Mode

Radio buttons toggle between light mode and dark mode. The selection is managed through `ThemeContext` and persisted in localStorage.

### Color-Based Theme

When enabled, a set of 4 color theme presets appears. Each preset changes accent colors across cards, buttons, and highlights. Themes are defined through CSS variables:

| Theme  | Colors                             |
| ------ | ---------------------------------- |
| Theme 1 | Defined by `--theme1-color1` through `--theme1-color4` |
| Theme 2 | Defined by `--theme2-color1` through `--theme2-color4` |
| Theme 3 | Defined by `--theme3-color1` through `--theme3-color4` |
| Theme 4 | Defined by `--theme4-color1` through `--theme4-color4` |

A "Reset to Default Theme" button restores the default color scheme.

### Implementation

The theme context provides:
- `theme` (light-mode / dark-mode)
- `colorTheme` (theme1 through theme4)
- `colorBasedTheme` (boolean toggle)
- `resetToDefaultTheme()` function

Preferences are persisted to localStorage under `colorTheme` and `colorBasedTheme` keys.

---

## UI Zoom Scaling

Users can adjust the UI size from 75% to 100% in 5% increments. The `ScaleContext` manages the current zoom level:

| Scale  | Label     |
| ------ | --------- |
| 100%   | Default   |
| 90%    | Slight    |
| 85%    | Compact   |
| 80%    | Tight     |
| 75%    | Minimal   |

Each option is rendered as a selectable card with a visual indicator showing which is active. The scale is persisted to localStorage under `uiScale` and applied through the `ScaleWrapper` component in the dashboard layout.

---

## Interview Talking Points

**On the theme context pattern:** "The theme, color theme, and zoom scale are managed through React context providers that wrap the dashboard layout. This avoids prop drilling and keeps preference management separate from business logic. Changes take effect immediately and are persisted to localStorage for cross-session persistence."

**On the 2FA toggle UX:** "The two-factor toggle uses an optimistic update pattern. The UI switches immediately while the API call is in flight. If the call fails, the toggle reverts and shows an error toast. This makes the settings page feel responsive even on slow connections."

## Related Documents

- [Settings Overview](overview.md) - Module structure
- [Account Settings](account-settings.md) - Amazon account management
- [UI Design System](../frontend/ui-design-system.md) - Theme and zoom architecture
- [Authentication Flow](../authentication/authentication-flow.md) - 2FA flow
