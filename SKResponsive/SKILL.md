---
name: sk-responsive
description: >-
  Guidelines and patterns for making Next.js pages, CSS modules, headers, pagination, and modals responsive.
---

# Next.js Responsiveness Skill
Guidelines and patterns for making Next.js pages, CSS modules, headers, pagination, and modals responsive.

## Overview
This skill outlines the process and best practices for converting desktop-first Next.js pages and components to mobile-friendly responsive designs using CSS Modules and media queries.

## Dependencies
None.

## Quick Start
To apply this skill:
1. First, ask the user if they would like to add an automation helper script to their workspace. Provide them with the following examples:
   - **CSS Scan Script**: A python utility to search CSS files for fixed absolute widths (`width: Xpx`) or layout patterns lacking media queries.
   - **Boilerplate Appender**: A script that appends standard responsiveness templates (e.g. `@media (max-width: 768px)`) directly to a list of CSS files.
   - **Responsive Audit Tool**: A script that lists all components and checks for mobile menu support or overflow properties.
2. If the user decides not to add a script, proceed with the manual workflow documented below.

## Workflow

### 1. Identify Elements and Layouts
- Locate non-responsive layouts in `.tsx` components and CSS modules.
- Check components like navigation headers, sidebars, page grids, card lists, pagination lists, and modals.
- Check if elements are clipped or overflowing on viewport widths under 768px.

### 2. Restructure Grid and Flex Containers
- Convert two-column/horizontal layouts to vertical layouts using media queries:
  ```css
  @media (max-width: 768px) {
    .layout {
      flex-direction: column;
      gap: 1.5rem;
    }
  }
  ```
- Change width boundaries from fixed pixel widths (`width: 250px`) to percentages or auto-resizing widths (`width: 100%`).
- **Dynamic Grid Rows on Mobile**: When a grid page size is fixed on desktop (e.g. 9 items, which is 3 rows of 3 columns) but is rendered on mobile in fewer columns (e.g. 2 columns or 1 column), avoid leaving it at 5 or 9 rows, which makes the page excessively long. Instead, implement a dynamic client-side `pageSize` adjustment to cap the display at exactly 3 rows:
  - Add a client-side resize listener in `useEffect` to safely compute page size on the client (preventing Next.js SSR hydration mismatches by initializing to desktop size and adjusting after mount):
    ```tsx
    const [pageSize, setPageSize] = useState(9);
    useEffect(() => {
      const handleResize = () => {
        if (window.innerWidth <= 375) {
          setPageSize(3); // 1 column * 3 rows
        } else if (window.innerWidth <= 768) {
          setPageSize(6); // 2 columns * 3 rows
        } else {
          setPageSize(9); // 3 columns * 3 rows
        }
      };
      handleResize();
      window.addEventListener('resize', handleResize);
      return () => window.removeEventListener('resize', handleResize);
    }, []);
    ```
  - Adjust grid styles media queries `min-height` accordingly (e.g. reduce mobile `min-height` from 920px to 560px for 3 rows) to prevent unnecessary blank spaces at the bottom.

### 3. Handle Long Lists and Overflowing Elements
- For horizontal version lists, tabs, or menus, convert them on mobile into horizontally scrollable pill containers:
  ```css
  @media (max-width: 768px) {
    .sidebarList {
      flex-direction: row;
      overflow-x: auto;
      white-space: nowrap;
    }
  }
  ```
- Hide browser scrollbars for clean visual layout using `-ms-overflow-style: none`, `scrollbar-width: none`, and `::-webkit-scrollbar { display: none; }`.

### 4. Optimize Pagination Elements
- Prevent pagination buttons from overflowing or wrapping awkwardly:
  - Reduce the active pagination page range (e.g., set `windowSize` logic in React state from 7 pages to 5).
  - Center pagination containers on mobile (`justify-content: center`).
  - Add `flex-wrap: wrap` and reduce button dimensions (`width: 28px`, `height: 28px`, `font-size: 0.85rem`) to save space.

### 5. Adapt Navigation Headers and Modals
- Replace complex user menu dropdowns and text elements with mobile-friendly links/tabs (such as a text-based bold "Cerrar sesión" tab).
- Use Next.js `<Image>` component instead of raw HTML images for optimization, and ensure mobile header icons remain visible by avoiding classes that have `display: none` under mobile queries.

### 6. Verify and Test
- Verify compile success using `npm run build` or inspect runtime dev compilation logs.
- Test the viewport adjustments down to 320px width.

## Common Mistakes
- **Server/Client Hydration Mismatch**: Hiding/showing elements using React state/JS window resizing instead of CSS media queries. Always prefer CSS-based display properties (`display: none` / `display: block`) for responsive visibility to keep Next.js SSR fully synchronized.
- **Missing Flex Wrap**: Failing to wrap elements in flex rows, causing elements to clip at small viewports.
- **Fixed Padding/Margins**: Keeping large desktop padding (`padding: 4rem`) on mobile. Reduce to `1rem` - `1.5rem`.
