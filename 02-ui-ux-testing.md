# Phase 2: UI/UX Testing 🎨

> **Objective:** Examine every screen, widget, and UI component for visual bugs,
> interaction problems, responsiveness failures, and accessibility violations.
> This is done through STATIC CODE ANALYSIS — reading the UI code to find problems
> that would appear on screen without needing an emulator.

---

## 🎯 WHY THIS PHASE MATTERS

80% of user-reported bugs are UI-related. Buttons that don't work, screens that overflow,
text that gets cut off, keyboards that cover inputs — these are the bugs users see FIRST.

---

## 📋 UI/UX CHECKS

### CHECK U1: Widget/Component Tree Analysis
```
ACTION: For each screen file, trace the widget tree from root to leaf
WHAT TO CHECK:
  ❑ Is the widget tree reasonable? (<15 nesting levels)
  ❑ Are there deeply nested conditional widgets?
  ❑ Are reusable components extracted (not copy-pasted)?
  ❑ Are widget keys used where needed? (lists, animated widgets)

FRAMEWORK-SPECIFIC:
  Flutter:
    ❑ Stateless vs Stateful used correctly?
    ❑ Are const constructors used for static widgets?
    ❑ Are widgets split into separate classes (not mega build methods)?
    ❑ Is the build method under 80 lines?

  React Native:
    ❑ Are functional components used (not class components)?
    ❑ Is React.memo used for expensive components?
    ❑ Are custom hooks extracted for reusable logic?
    ❑ Is JSX clean and readable?

  Kotlin/Compose:
    ❑ Are @Composable functions small and focused?
    ❑ Is remember/rememberSaveable used correctly?
    ❑ Are side effects properly managed (LaunchedEffect, DisposableEffect)?

  SwiftUI:
    ❑ Are views small and use proper composition?
    ❑ Is @State vs @StateObject vs @ObservedObject used correctly?
    ❑ Are view modifiers ordered properly?

COMMON BUGS:
  🐛 Mega build method with 500+ lines (unreadable, unmaintainable)
  🐛 Widget tree 20+ levels deep (performance nightmare)
  🐛 Copy-pasted UI code instead of extracted components
  🐛 Missing keys on list items (causes state bugs on reorder)

CITATION REQUIRED: Show the most complex widget tree and assess it
```

### CHECK U2: Responsive Design & Screen Adaptation
```
WHAT TO CHECK:
  ❑ Are dimensions hardcoded in pixels? (BAD — should be relative)
  ❑ Is there adaptation for different screen sizes?
  ❑ Are there MediaQuery / LayoutBuilder / Flex usage?
  ❑ Is text scalable? (respects system font size settings)
  ❑ Is landscape orientation handled? (or properly locked to portrait)
  ❑ Are there overflow risks? (Row without Expanded, long text without overflow)
  ❑ Is there tablet support? (or at least doesn't break on tablets)
  ❑ Is notch/safe area handled? (SafeArea widget, safe area insets)

RED FLAGS:
  🔴 Container(width: 375) — hardcoded iPhone width
  🔴 SizedBox(height: 812) — hardcoded screen height
  🔴 Text without maxLines or overflow property in constrained space
  🔴 Row with multiple children but no Flexible/Expanded
  🔴 No SafeArea wrapper on screens
  🔴 Padding(padding: EdgeInsets.only(top: 44)) — hardcoded status bar height
  🔴 Fixed pixel sizes for fonts instead of scalable units

COMMON OVERFLOW SCENARIOS:
  - Long user names that break layout
  - RTL languages that reverse layout
  - Large system font size (accessibility)
  - Small screens (320px width)
  - Tablets (768px+ width)
  - Foldable devices
  - Landscape orientation

CITATION REQUIRED: Show at least 3 hardcoded dimension instances (or confirm none exist)
```

### CHECK U3: Button & Interaction Testing
```
WHAT TO CHECK FOR EVERY INTERACTIVE ELEMENT:
  ❑ Does every button have an onPressed/onTap handler?
  ❑ Is the handler actually doing something? (not empty () => {} )
  ❑ Is there visual feedback on press? (ripple, opacity change, animation)
  ❑ Is double-tap prevented? (prevents double form submission)
  ❑ Is the touch target at least 48x48 dp/pt? (accessibility requirement)
  ❑ Are disabled states handled? (greyed out when action unavailable)
  ❑ Is there a loading state during async operations?

COMMON BUGS:
  🐛 Button with onPressed: null but no visual disabled state
  🐛 Button calls async function but no loading indicator
  🐛 Button can be tapped multiple times firing multiple API calls
  🐛 GestureDetector with no visual feedback (user doesn't know they tapped)
  🐛 Button text too long for button width (overflow/clip)
  🐛 onPressed handler navigates but doesn't await result
  🐛 IconButton without tooltip (accessibility)

FLUTTER-SPECIFIC:
  ❑ ElevatedButton vs TextButton vs OutlinedButton used appropriately?
  ❑ InkWell/GestureDetector has sufficient touch area?
  ❑ Button is not wrapped in GestureDetector AND has onPressed (double handler)

REACT NATIVE-SPECIFIC:
  ❑ TouchableOpacity vs Pressable used correctly?
  ❑ Is hitSlop set for small buttons?
  ❑ Are buttons inside ScrollView properly handled?

CITATION REQUIRED: List ALL buttons found per screen with their handler status
```

### CHECK U4: Form & Input Validation
```
WHAT TO CHECK:
  ❑ Do all TextFields/TextInputs have proper keyboard types?
    - Email → TextInputType.emailAddress
    - Phone → TextInputType.phone
    - Password → obscureText: true
    - Number → TextInputType.number
  ❑ Is there real-time validation? (not just on submit)
  ❑ Are error messages clear and specific?
  ❑ Is the form scrollable when keyboard appears?
  ❑ Is there input length limitation? (maxLength)
  ❑ Are special characters handled? (emoji, RTL text, unicode)
  ❑ Is paste functionality working correctly?
  ❑ Is autofill supported? (username, password, email)
  ❑ Is there a "show password" toggle?
  ❑ Is the submit button disabled until form is valid?

COMMON BUGS:
  🐛 Email field accepts "notanemail" without validation
  🐛 Password field has no minimum length requirement
  🐛 Keyboard covers input field (no scroll adjustment)
  🐛 Form loses data on screen rotation
  🐛 No error message shown — field just turns red
  🐛 Validation only runs on submit, not on field change
  🐛 Phone number field doesn't accept country codes
  🐛 Text field allows SQL injection characters without sanitization
  🐛 No maximum input length (can paste 10MB text)

VALIDATION RULES TO CHECK:
  - Email: valid format with @ and domain
  - Phone: country-appropriate format
  - Password: length, complexity, match confirmation
  - Name: no numbers or special characters (if applicable)
  - Amount: no negative, proper decimal handling
  - Date: valid date range, no future dates (for birthdate)

CITATION REQUIRED: Show every form and its validation logic
```

### CHECK U5: Navigation & Screen Flow
```
WHAT TO CHECK:
  ❑ Can the user reach every screen from the expected entry point?
  ❑ Does the back button work correctly on every screen?
  ❑ Is the navigation history clean? (no duplicate screens in stack)
  ❑ Do bottom tabs maintain their state when switching?
  ❑ Is there a splash screen and does it transition smoothly?
  ❑ Is there a proper loading state while data fetches on screen entry?
  ❑ Are transitions smooth and appropriate?
  ❑ Does pull-to-refresh work where expected?

DEAD-END DETECTION:
  - Find screens with no back button AND no navigation elements
  - Find error states with no retry or back option
  - Find success states with no "what next" action
  - Find empty states with no guidance

COMMON BUGS:
  🐛 Login success → Home screen → Back button → Login again (should not happen)
  🐛 Push notification opens wrong screen
  🐛 Tab bar disappears on nested navigation
  🐛 Modal can't be dismissed (no X button, no swipe-to-dismiss)
  🐛 Screen shows stale data from previous visit (no refresh)
  🐛 Circular navigation: Screen A → Screen B → Screen A (infinite loop possible)

CITATION REQUIRED: Map the navigation graph (which screens connect to which)
```

### CHECK U6: List & Scroll Performance
```
WHAT TO CHECK:
  ❑ Are long lists using lazy loading? (ListView.builder, FlatList, RecyclerView)
  ❑ Is pagination implemented for large datasets?
  ❑ Are list items optimized? (no rebuild on scroll)
  ❑ Is there an empty state when list has no items?
  ❑ Is there a loading state while list data loads?
  ❑ Is there error handling if list data fails to load?
  ❑ Are images in lists cached and loaded lazily?
  ❑ Is there pull-to-refresh on the list?
  ❑ Is scroll position preserved on navigation and return?

RED FLAGS:
  🔴 ListView(children: [...]) with 100+ items (loads ALL at once)
  🔴 Column + SingleChildScrollView wrapping list items
  🔴 No pagination — fetches 10,000 records at once
  🔴 Image.network inside list without caching
  🔴 Complex calculations inside list item build

FRAMEWORK-SPECIFIC:
  Flutter:
    ❑ ListView.builder used instead of ListView(children)?
    ❑ SliverList/SliverGrid for complex scroll layouts?
    ❑ AutomaticKeepAliveClientMixin for tab views?

  React Native:
    ❑ FlatList used instead of ScrollView with map()?
    ❑ keyExtractor properly set?
    ❑ getItemLayout provided for fixed-height items?
    ❑ removeClippedSubviews enabled for large lists?

CITATION REQUIRED: Show every list implementation and its loading strategy
```

### CHECK U7: Image & Media Handling
```
WHAT TO CHECK:
  ❑ Are images loaded with proper placeholder/loading state?
  ❑ Are error states handled for failed image loads?
  ❑ Is image caching implemented?
  ❑ Are images properly sized? (not loading 4K images for 100x100 thumbnails)
  ❑ Are image formats appropriate? (WebP preferred, SVG for icons)
  ❑ Is there memory-safe handling for galleries?
  ❑ Are videos loaded with proper controls?

COMMON BUGS:
  🐛 Image fails to load → shows empty space or red error box
  🐛 Loading full resolution image for a 50x50 avatar
  🐛 No caching — re-downloads images on every screen visit
  🐛 Memory crash when scrolling through image gallery
  🐛 Dark mode shows images with white backgrounds
  🐛 Circular avatar doesn't clip properly
  🐛 Network images block the UI thread

CITATION REQUIRED: Show image loading code and caching strategy
```

### CHECK U8: Accessibility (a11y)
```
WHAT TO CHECK:
  ❑ Do all images have content descriptions (semantics/alt text)?
  ❑ Are touch targets at least 48x48 dp?
  ❑ Is color contrast sufficient? (4.5:1 for text, 3:1 for large text)
  ❑ Is the app usable with screen reader? (TalkBack/VoiceOver)
  ❑ Are interactive elements properly labeled?
  ❑ Does the app respect system font size settings?
  ❑ Are there visual indicators beyond color? (not just red/green)
  ❑ Is focus order logical for keyboard/switch navigation?

COMMON VIOLATIONS:
  🐛 IconButton without semanticLabel/accessibilityLabel
  🐛 Image without Semantics wrapper or contentDescription
  🐛 Light gray text on white background (low contrast)
  🐛 Information conveyed only through color (colorblind users can't see)
  🐛 Custom widgets not properly exposed to accessibility tree
  🐛 Text doesn't scale with system settings (hardcoded font sizes)

CITATION REQUIRED: Check at least 5 interactive elements for accessibility compliance
```

### CHECK U9: Dark Mode & Theming
```
WHAT TO CHECK:
  ❑ Is dark mode supported?
  ❑ Does the app use theme colors (not hardcoded colors)?
  ❑ Are all screens tested for both light and dark mode?
  ❑ Are images/icons appropriate for both themes?
  ❑ Is system theme preference respected?
  ❑ Can user toggle theme manually?
  ❑ Is the theme consistent across ALL screens?

COMMON BUGS:
  🐛 Text color hardcoded as black → invisible on dark background
  🐛 Container with hardcoded white background → blinding in dark mode
  🐛 Icons designed for light theme → invisible in dark theme
  🐛 Status bar color doesn't change with theme
  🐛 Some screens use theme, others use hardcoded colors
  🐛 Shadow effects look wrong in dark mode

HOW TO CHECK WITHOUT EMULATOR:
  - Search for hardcoded color values: Colors.black, Colors.white, #000000, #FFFFFF
  - Search for Color(0xFF...) patterns instead of Theme.of(context)
  - Check if ThemeData or equivalent is defined and used consistently
  - Look for conditional theme checks (if dark mode → else →)

CITATION REQUIRED: Show theme setup and at least 3 instances of color usage
```

### CHECK U10: Localization & Internationalization (i18n)
```
WHAT TO CHECK:
  ❑ Are all user-facing strings externalized? (not hardcoded)
  ❑ Is RTL layout supported? (for Arabic, Hebrew, etc.)
  ❑ Are date/time formats locale-appropriate?
  ❑ Are number formats locale-appropriate? (decimal comma vs period)
  ❑ Are currency formats correct?
  ❑ Are pluralization rules handled?
  ❑ Is the app language switchable?

RED FLAGS:
  🔴 Text('Welcome') instead of localization key
  🔴 DateFormat('MM/dd/yyyy') hardcoded (not all countries use this)
  🔴 String concatenation for translatable text
  🔴 No RTL testing for layouts
  🔴 Translated text may be 2-3x longer than English (German, Finnish)

CITATION REQUIRED: Count hardcoded strings vs localized strings
```

---

## 🚦 PHASE 2 GATE — MANDATORY CHECKLIST

```
PHASE 2 GATE CHECKLIST:
  □ [U1]  Widget/component tree analyzed for all screens
  □ [U2]  Responsive design checked (hardcoded dims, overflow risks)
  □ [U3]  All buttons/interactions verified (handlers, feedback, states)
  □ [U4]  All forms/inputs validated (keyboard types, validation rules)
  □ [U5]  Navigation flow mapped (dead-ends, back behavior)
  □ [U6]  List/scroll implementations checked (lazy loading, pagination)
  □ [U7]  Image/media handling verified (caching, error states, sizing)
  □ [U8]  Accessibility compliance checked (labels, contrast, targets)
  □ [U9]  Dark mode/theming verified (hardcoded colors found)
  □ [U10] Localization checked (hardcoded strings counted)
  □ Minimum 8 code citations provided
  □ Files examined list produced
  □ Screen-by-screen summary produced
```

### Gate Report Format:
```
═══════════════════════════════════════════════════════════
  ✅ PHASE 2 COMPLETE: UI/UX Testing
  📊 Findings: [X] Critical | [Y] High | [Z] Medium | [W] Low
  📋 Gate Status: [PASSED/FAILED] ([checked]/13 items)

  🎨 UI Health Score: [X/10]
  Screens Analyzed: [count]
  Buttons Verified: [count]
  Forms Checked: [count]
  Accessibility Score: [X/10]
  Responsive Design: [Good/Fair/Poor]
═══════════════════════════════════════════════════════════
```

### ⛔ STOP POINT
**Present all UI findings to user before proceeding to Phase 3.**
