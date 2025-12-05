# CLAUDE.md - Fundus PACS Viewer

## Project Overview

**Fundus PACS** is a medical imaging viewer specifically designed for fundus photography (retinal imaging). This is a standalone, single-file web application for viewing and analyzing eye fundus images with support for bilateral comparison (both eyes simultaneously).

**Project Type**: Medical Imaging Viewer (Ophthalmology)
**Primary Language**: JavaScript (React via CDN)
**Architecture**: Single-page application (SPA) in standalone HTML file
**Medical Context**: PACS = Picture Archiving and Communication System

## Repository Structure

```
FUNDUS_PACS/
├── FUNDUS_PACS.html    # Main application (standalone React app)
├── README.md           # Basic project description (Chinese)
└── CLAUDE.md           # This file - AI assistant guide
```

### Key Files

- **FUNDUS_PACS.html**: Complete standalone application containing all HTML, CSS, and JavaScript
  - No build process required
  - Can be opened directly in a browser
  - All dependencies loaded via CDN

## Technical Stack

### Dependencies (CDN-based)
- **React 18** (`react.development.js`, `react-dom.development.js`)
- **Tailwind CSS** (via CDN)
- **Babel Standalone** (for JSX transformation in browser)
- **Lucide React** (icon library, v0.468.0)

### No Build System
This project intentionally uses a standalone HTML approach:
- No npm/package.json
- No build tools (webpack, vite, etc.)
- No transpilation step
- Direct browser execution

## Medical Imaging Conventions

### File Naming Convention (CRITICAL)

The application expects fundus image files to follow this naming pattern:

```
{MRN}_{YYYYMMDD}_{HHMMSS}_{Type}_{Side}_{Sequence}.{ext}
```

**Example**: `01424_20251202_080330_Color_R_001.jpg`

**Components**:
- `MRN`: Medical Record Number (patient ID)
- `YYYYMMDD`: Date of imaging (8 digits)
- `HHMMSS`: Time of imaging (6 digits)
- `Type`: Image type (e.g., "Color")
- `Side`: Eye laterality
  - `R` or `OD` = Right eye (Oculus Dexter)
  - `L` or `OS` = Left eye (Oculus Sinister)
- `Sequence`: Image sequence number (e.g., 001, 002)
- Extension: `.jpg`, `.jpeg`, or `.png`

**Parsing Logic** (FUNDUS_PACS.html:106-133):
```javascript
const parseFilename = (filename) => {
    const parts = nameOnly.split('_');
    if (parts.length < 5) return null;

    const mrn = parts[0];
    const dateStr = parts[1]; // YYYYMMDD
    const timeStr = parts[2];
    const sideCode = parts[4];

    const side = (sideCode === 'R' || sideCode === 'OD') ? 'OD' : 'OS';
    return { mrn, date, side, ... };
};
```

### Medical Terminology

- **OD** (Oculus Dexter): Right eye
- **OS** (Oculus Sinister): Left eye
- **OU** (Oculus Uterque): Both eyes
- **Fundus**: The interior surface of the eye opposite to the lens
- **Red-Free Imaging**: Green-filtered imaging technique highlighting retinal vessels

## Application Architecture

### Component Structure

The application follows a standard React component hierarchy:

```
App (Main Container)
├── Sidebar
│   ├── File import button
│   ├── Search box
│   └── Grouped file list (by date)
├── Toolbar
│   ├── View mode toggles (single/split)
│   ├── Tool selection (pan/adjust)
│   ├── Image adjustment sliders
│   └── Filter controls (red-free, invert)
└── Main Viewport
    └── ImageCanvas (1-2 instances depending on view mode)
```

### Key Components

#### 1. Sidebar (FUNDUS_PACS.html:137-285)
**Purpose**: File browsing and patient search
- Accepts local folder import
- Searches by MRN (patient ID)
- Groups images by date
- Supports double-click to open images
- Double-click on date header opens bilateral view (OU)

#### 2. Toolbar (FUNDUS_PACS.html:287-408)
**Purpose**: Image manipulation controls
- View mode: Single vs. Split (bilateral)
- Active tool: Pan vs. Adjust (W/L)
- Brightness/Contrast sliders
- Red-Free filter toggle
- Invert filter toggle
- Zoom controls
- Reset button

#### 3. ImageCanvas (FUNDUS_PACS.html:410-526)
**Purpose**: Individual image display and interaction
- Mouse wheel zoom
- Pan tool: Click and drag to move image
- Adjust tool: Vertical drag = brightness, Horizontal drag = contrast
- Displays patient metadata (MRN, date, laterality)
- Handles missing images with placeholder

#### 4. App (Main) (FUNDUS_PACS.html:528-670)
**Purpose**: Application state management
- Manages file list
- Controls view mode (single/split)
- Manages image settings (shared across both eyes)
- Handles file selection logic

### State Management

The application uses React hooks for state management:

```javascript
// Main App State
const [allFiles, setAllFiles] = useState(DEFAULT_FILES);
const [selectedFiles, setSelectedFiles] = useState([]);
const [viewMode, setViewMode] = useState('single');
const [activeTool, setActiveTool] = useState('pan');

// Image Settings (shared between both eyes)
const [imageSettings, setImageSettings] = useState({
    brightness: 100,
    contrast: 100,
    zoom: 1,
    invert: false,
    redFree: false
});
```

**Important**: Image settings are synchronized across both eyes when in split view. This is intentional for medical comparison purposes.

## Key Features

### 1. File Import
- Browser folder selection via `<input webkitdirectory>`
- Filters for image files (.jpg, .jpeg, .png)
- Automatically parses filenames to extract metadata

### 2. View Modes
- **Single**: One image at a time
- **Split (OU)**: Side-by-side bilateral comparison

### 3. Image Manipulation
- **Zoom**: Mouse wheel or buttons (10%-500%)
- **Pan**: Click and drag (pan tool active)
- **Brightness/Contrast**: Click and drag (adjust tool) or sliders
- **Red-Free Filter**: Digital green channel filter (SVG filter)
- **Invert**: Color inversion

### 4. Smart Pairing
When selecting an image in split mode:
- Automatically finds contralateral eye from same date
- Displays OD (right) on left, OS (left) on right (standard medical convention)

### 5. Mock Data
Default mock data for demonstration (FUNDUS_PACS.html:87-101):
- 3 sample patients (MRN: 01424, 09932, 10234)
- 3 dates each
- 4 images per date (2 per eye)

## Development Workflow

### Testing the Application

1. **Open in Browser**:
   ```bash
   # Simply open the HTML file
   open FUNDUS_PACS.html
   # or on Linux
   xdg-open FUNDUS_PACS.html
   ```

2. **Test with Mock Data**:
   - Search for "01424" in the sidebar
   - Double-click a date header to open bilateral view
   - Test image controls

3. **Test with Real Data**:
   - Click "載入本機資料夾" (Load Local Folder)
   - Select a folder containing properly named fundus images
   - Verify filename parsing works correctly

### Making Changes

#### Editing the Application

Since this is a standalone HTML file, all changes are made to `FUNDUS_PACS.html`:

1. **JavaScript/React Code**: Located in `<script type="text/babel">` block (lines 66-674)
2. **Styles**:
   - Inline `<style>` block (lines 21-48)
   - Tailwind utility classes in JSX
3. **SVG Filters**: Defined in hidden SVG block (lines 53-64)

#### Code Organization Within the File

```
FUNDUS_PACS.html
├── <head>
│   ├── Meta tags
│   ├── CDN script imports
│   └── Custom CSS
├── <body>
│   ├── <div id="root"> (React mount point)
│   ├── SVG filter definitions
│   └── <script type="text/babel">
│       ├── Imports and setup
│       ├── Mock data generation
│       ├── Helper functions (parseFilename)
│       ├── Component: Sidebar
│       ├── Component: Toolbar
│       ├── Component: ImageCanvas
│       ├── Component: App
│       └── ReactDOM.render()
└── </body>
```

### Adding New Features

When adding features, follow this pattern:

1. **Add state to App component** if it affects multiple components
2. **Pass props down** to child components
3. **Use callbacks** to communicate back up to parent
4. **Keep image settings synchronized** in split view mode

Example - Adding a new filter:
```javascript
// 1. Add to imageSettings state
const [imageSettings, setImageSettings] = useState({
    // ... existing settings
    newFilter: false  // Add here
});

// 2. Add UI control in Toolbar
<button
    onClick={() => updateSetting('newFilter', !settings.newFilter)}
>
    New Filter
</button>

// 3. Apply in ImageCanvas filterStyle
const filterStyle = {
    filter: `
        ...existing filters...
        ${settings.newFilter ? 'sepia(1)' : ''}
    `,
    // ...
};
```

## Common Modification Patterns

### Changing Image Processing

All image filters are applied via CSS filters in `ImageCanvas` component:

```javascript
const filterStyle = {
    filter: `
        brightness(${settings.brightness}%)
        contrast(${settings.contrast}%)
        invert(${settings.invert ? 1 : 0})
        ${settings.redFree ? 'url(#red-free-filter)' : ''}
    `,
    // ...
};
```

**Available CSS Filters**:
- `brightness()`, `contrast()`, `saturate()`
- `hue-rotate()`, `invert()`, `sepia()`
- `blur()`, `grayscale()`
- Custom SVG filters via `url(#filter-id)`

### Modifying File Name Parsing

Edit the `parseFilename()` function (lines 106-133) if your naming convention differs:

```javascript
const parseFilename = (filename) => {
    // Modify parsing logic here
    // Must return object with: mrn, date, side, filename, id
};
```

### Changing Default Settings

Modify initial state in App component (lines 535-541):

```javascript
const [imageSettings, setImageSettings] = useState({
    brightness: 100,  // Change defaults here
    contrast: 100,
    zoom: 1,
    invert: false,
    redFree: false
});
```

## Security Considerations

### Medical Data Privacy

**IMPORTANT**: This application processes medical images locally in the browser.

- **No server uploads**: All processing happens client-side
- **No data persistence**: Files are loaded via `FileReader` API and stored in memory
- **Object URLs cleaned up**: `URL.revokeObjectURL()` called on unmount
- **No external API calls**: Application runs completely offline after initial load

### HIPAA Compliance Notes

While this application doesn't transmit data, consider:
- Browser caching may store images
- Browser history may log filenames (which contain MRN)
- Screenshots may capture PHI
- Recommend using in private/incognito mode for sensitive data

## Browser Compatibility

### Tested Browsers
- Chrome/Edge 90+ (recommended)
- Firefox 88+
- Safari 14+

### Required Features
- ES6+ JavaScript support
- CSS Grid
- Flexbox
- FileReader API
- Input `webkitdirectory` attribute (for folder selection)

### Known Limitations
- Folder selection not supported in older browsers
- Large image sets (>100 images) may cause performance issues
- Mobile browsers may have limited memory for large images

## Keyboard Shortcuts

Currently, no keyboard shortcuts are implemented. Potential additions:

- `R`: Reset all settings
- `Space`: Toggle between pan/adjust tools
- `1/2`: Switch between single/split view
- `+/-`: Zoom in/out
- Arrow keys: Pan image

## Troubleshooting

### Images Not Loading

1. **Check filename format**: Must match `MRN_DATE_TIME_TYPE_SIDE_SEQ.ext`
2. **Verify file extension**: Must be `.jpg`, `.jpeg`, or `.png`
3. **Check console**: Open browser DevTools to see parsing errors

### Search Not Working

- Mock data only includes MRNs: 01424, 09932, 10234
- After importing real files, search filters by MRN substring match

### Bilateral View Missing Contralateral Eye

- Ensure both eyes exist with matching MRN and date
- Verify side codes are parsed correctly (R/OD vs L/OS)
- Check that files are from same imaging session (same date)

## Future Enhancement Ideas

### Potential Features
- Keyboard shortcuts
- Image export with current settings applied
- Annotation tools (markers, measurements)
- Comparison mode (same eye, different dates)
- DICOM support
- Server integration for image storage
- Multi-user support
- Image quality metrics
- Automatic lesion detection (AI/ML)

### Architecture Improvements
- Split into modular files with build system
- Add TypeScript for type safety
- Implement proper state management (Redux/Zustand)
- Add unit tests
- Progressive Web App (PWA) support
- Offline caching

## Git Workflow

### Branching Strategy
- Work on feature branches starting with `claude/`
- Branch naming: `claude/claude-md-{session-id}`
- Never commit directly to main

### Commit Message Format
```
<type>: <short description>

<optional longer description>
```

**Types**: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`

**Examples**:
```
feat: add keyboard shortcuts for common operations
fix: correct bilateral pairing for mixed laterality codes
docs: update CLAUDE.md with deployment instructions
```

### Before Committing

1. Test in at least 2 browsers
2. Verify mock data still works
3. Check for console errors
4. Ensure HTML file is valid

## Deployment

### Hosting Options

**Option 1: Static Hosting**
- GitHub Pages
- Netlify
- Vercel
- Any static file server

**Option 2: Local Network**
```bash
# Simple Python server
python3 -m http.server 8000

# Access at http://localhost:8000/FUNDUS_PACS.html
```

**Option 3: Intranet**
- Deploy to hospital/clinic intranet
- Ensure HIPAA compliance measures
- Consider access controls

### Production Considerations

Before production deployment:

1. **Switch to production React builds**:
   ```html
   <!-- Replace development with production -->
   <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
   <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
   ```

2. **Remove mock data** or make it clearly labeled as demo

3. **Add disclaimer** about medical use

4. **Test with real data** from target environment

5. **Document** exact filename convention expected

## AI Assistant Guidelines

### When Modifying This Codebase

1. **Always read the full file** before making changes - it's a single file
2. **Test changes** by suggesting user open in browser
3. **Preserve medical conventions** (OD/OS, OU terminology)
4. **Maintain filename parsing compatibility** - changes here affect all users
5. **Keep image settings synchronized** in split view mode
6. **Don't add build complexity** - maintain standalone nature unless explicitly requested
7. **Consider medical context** - this is a diagnostic tool

### Code Style Preferences

- **React**: Functional components with hooks
- **Naming**: camelCase for JS, kebab-case for CSS classes
- **Comments**: Explain medical/domain logic, not obvious code
- **Tailwind**: Use utility classes, minimal custom CSS
- **File organization**: Keep logical sections together in the HTML file

### Medical Domain Knowledge

When working on this codebase, understand:
- **Right eye (OD) displays on LEFT** side in bilateral view (radiological convention)
- **Left eye (OS) displays on RIGHT** side in bilateral view
- **Red-free imaging** helps visualize retinal nerve fiber layer
- **Brightness/Contrast** (Window/Level in medical imaging) critical for diagnosis
- **Zoom and pan** are essential for detail examination

### Questions to Ask User

Before making significant changes:
- "What is your exact filename convention?"
- "Do you need DICOM support?"
- "Should this remain a standalone file or move to a build system?"
- "Are there specific medical standards we need to follow?"
- "What is your deployment target? (web server, intranet, offline)"

## References

### Medical Imaging Standards
- DICOM: Digital Imaging and Communications in Medicine
- PACS: Picture Archiving and Communication System
- HL7: Health Level Seven (for EHR integration)

### Related Technologies
- React Documentation: https://react.dev
- Tailwind CSS: https://tailwindcss.com
- Lucide Icons: https://lucide.dev

## Contact & Support

This is an open-source project. For issues or questions:
- Check the README.md for basic info
- Review this CLAUDE.md for detailed documentation
- Test with mock data first before reporting issues

---

**Last Updated**: 2025-12-05
**Version**: 1.0
**Maintained by**: AI Assistants & Contributors
