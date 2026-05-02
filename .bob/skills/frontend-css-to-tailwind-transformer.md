# CSS to Tailwind Transformer

Transforms existing CSS styling into Tailwind CSS utility classes while preserving the visual appearance and functionality. Use this when migrating legacy CSS to Tailwind, refactoring inline styles, or converting CSS modules to utility-first approach. The transformation maintains responsive design, pseudo-classes, and complex selectors while following Tailwind best practices.

**Category:** frontend
**Roles:** frontend, fullstack

---

## Prerequisites

- Tailwind CSS installed and configured in the project (`tailwind.config.js` exists)
- Access to the CSS file or component with inline styles to be transformed
- Node.js environment with npm or yarn available
- Basic understanding of Tailwind utility classes and naming conventions

---

## Inputs

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `source_path` | string | Yes | Path to the CSS file or component file containing styles to transform |
| `target_format` | string | No | Output format: `inline` (JSX className), `separate` (new CSS file), or `both` (default: `inline`) |
| `preserve_custom` | boolean | No | Whether to preserve custom CSS that cannot be converted to Tailwind (default: `true`) |
| `responsive_breakpoints` | array | No | Custom breakpoints to use (default: Tailwind's sm, md, lg, xl, 2xl) |
| `output_path` | string | No | Custom output path for transformed file (default: same directory with `.tailwind` suffix) |

---

## Steps

### Step 1: Read Source File and Tailwind Config

First, use the `read_file` tool to read both the source file at the path provided by the user and the `tailwind.config.js` file together.
Analyze the CSS rules, selectors, properties, and values that need to be transformed.
Note any custom colors, spacing values, or breakpoints defined in the Tailwind config that should be used in the transformation.

### Step 2: Parse and Categorize CSS Rules

Next, analyze the CSS content to categorize rules into groups:
- Layout properties (display, position, flexbox, grid)
- Spacing properties (margin, padding, gap)
- Typography properties (font-size, font-weight, line-height, text-align)
- Color properties (color, background-color, border-color)
- Border and shadow properties (border, border-radius, box-shadow)
- Responsive media queries and breakpoints
- Pseudo-classes and pseudo-elements (:hover, :focus, ::before, ::after)
- Complex selectors that may need special handling

Identify any CSS that cannot be directly converted to Tailwind utilities and should be preserved as custom CSS.

### Step 3: Map CSS Properties to Tailwind Classes

Then, create a mapping of each CSS property to its equivalent Tailwind utility class:
- Convert pixel values to Tailwind spacing scale (px-4, mt-8, etc.)
- Map color values to Tailwind color palette or custom colors from config
- Transform flexbox/grid properties to Tailwind utilities (flex, grid, items-center, etc.)
- Convert responsive media queries to Tailwind breakpoint prefixes (sm:, md:, lg:)
- Map pseudo-classes to Tailwind variants (hover:, focus:, active:)
- Handle complex properties like box-shadow, gradients, and transforms

Use the Tailwind documentation patterns for accurate class naming.

### Step 4: Generate Tailwind Class Strings

After that, construct the complete Tailwind class strings for each element:
- Combine all utility classes in a logical order (layout → spacing → typography → colors → effects)
- Apply responsive variants using breakpoint prefixes
- Add state variants (hover, focus, active) where applicable
- Group related utilities together for readability
- Remove duplicate or conflicting classes

Ensure the class strings are properly formatted and follow Tailwind's recommended class ordering.

### Step 5: Handle Custom CSS and Edge Cases

Next, identify CSS rules that cannot be converted to Tailwind utilities:
- Complex animations and keyframes
- Custom pseudo-elements with content
- Advanced CSS features (clip-path, filters with multiple values)
- Browser-specific prefixes (-webkit-, -moz-)

If `preserve_custom` is true, extract these rules into a separate custom CSS section with descriptive comments.
Consider using Tailwind's `@apply` directive for frequently repeated custom patterns.

### Step 6: Transform the Source File

Then, use the `apply_diff` tool to update the source file:
- Replace CSS class definitions with Tailwind utility classes
- Update className attributes in JSX/HTML with the generated Tailwind classes
- Remove the old CSS imports or style blocks
- Add comments indicating the transformation for complex conversions

If `target_format` is `separate`, use `write_to_file` to create a new file with the Tailwind classes.
Maintain proper indentation and formatting for readability.

### Step 7: Create Custom CSS File (if needed)

If there are preserved custom styles, use the `write_to_file` tool to create a custom CSS file at `{output_path}.custom.css`.
Include only the CSS rules that could not be converted to Tailwind utilities.
Add clear comments explaining why each rule was preserved and suggestions for potential Tailwind alternatives.
Import this file in the component or main CSS file.

### Step 8: Update Tailwind Config (if needed)

If the transformation revealed custom values not in the default Tailwind config (colors, spacing, fonts), use the `read_file` tool to check `tailwind.config.js`.
Then use the `apply_diff` tool to extend the theme with these custom values:
- Add custom colors to `theme.extend.colors`
- Add custom spacing to `theme.extend.spacing`
- Add custom fonts to `theme.extend.fontFamily`

This ensures the generated Tailwind classes will work correctly.

### Step 9: Verify Visual Consistency

Finally, use the `execute_command` tool to run the development server or build process:
```
npm run dev
```
or
```
npm run build
```

Verify that the transformed styles render correctly and match the original visual appearance.
Check responsive behavior at different breakpoints and test interactive states (hover, focus).
If there are visual discrepancies, use `read_file` to examine the generated classes and `apply_diff` to make corrections.

---

## Outputs

- `{source_path}` — Updated file with Tailwind utility classes replacing original CSS
- `{output_path}.custom.css` — (Optional) Custom CSS file for rules that couldn't be converted
- `tailwind.config.js` — (If updated) Extended config with custom theme values
- Transformation report — Summary of converted classes, preserved custom CSS, and any warnings

---

## Example Usage

**User request:**
> Transform the CSS in `components/Button.css` to Tailwind classes. The button has hover effects, responsive sizing, and custom shadow.

**Expected output:**
- `components/Button.jsx` — Updated with Tailwind classes like `px-6 py-3 bg-blue-500 hover:bg-blue-600 text-white font-semibold rounded-lg shadow-lg transition-colors duration-200 sm:px-4 sm:py-2 md:px-6 md:py-3`
- `components/Button.custom.css` — (If needed) Custom box-shadow or animation that couldn't be converted
- Transformation report showing 95% of CSS successfully converted to Tailwind utilities

---

## Notes

- Tailwind's default spacing scale uses 0.25rem (4px) increments; adjust pixel values accordingly
- Use Tailwind's arbitrary values `[value]` syntax for one-off custom values (e.g., `w-[347px]`)
- Consider using `@apply` in CSS for frequently repeated utility combinations
- Maintain semantic class names for complex components even when using Tailwind
- Use Tailwind's `group` and `peer` utilities for parent-child state relationships

## Warnings

> ⚠️ Complex CSS animations and keyframes cannot be directly converted to Tailwind utilities. These will be preserved as custom CSS or need manual conversion using Tailwind's animation utilities.

> ⚠️ CSS specificity may change after transformation. Test thoroughly to ensure styles are applied correctly, especially in components with nested selectors.

> ⚠️ Some CSS features like `calc()`, CSS variables, and advanced selectors may require custom CSS or Tailwind config extensions. Review the transformation report for these cases.

> ⚠️ Always verify responsive behavior after transformation, as media query breakpoints may not map exactly to Tailwind's default breakpoints.

## Related Skills

- Tailwind Config Generator
- Component Refactoring
- CSS Optimization
- Responsive Design Converter