# Project Bookclub

### A Smart Book Selection Wheel Powered by Google Sheets

**By: Robert Cruz**

---

## Overview

Project Bookclub is an interactive web application that helps book clubs and individual readers randomly select their next read. It combines a visually appealing spinning wheel with powerful Google Sheets integration, allowing you to manage your TBR (To Be Read) list dynamically and track your reading journey.

---

## Features

### Google Sheets Integration
- **Sync Full TBR** - Load your entire book list from Column A of your Google Sheet
- **Sync Wheel Picks** - Load only books marked for the wheel in Column G
- **Smart Filtering** - Automatically skip books with checked checkboxes in Column E
- **Real-time Updates** - One-click sync to pull the latest changes from your sheet
- **Custom List Detection** - Automatically detects when you manually edit the book list and shows "Custom" status

### Interactive Wheel
- **Spin to Select** - Click and hold to charge, release to spin
- **Dynamic Sizing** - Automatically adjusts text size and placement based on the number of books
- **Visual Feedback** - Clear display of the winning book with confetti celebration, and the winning slice is spotlighted
- **Color-Coded Picks** - Each person's picks get their own slice color, with a legend showing how many books each person has on the wheel
- **Touch Support** - Works on both desktop and mobile devices

### Book Cover Integration
- **Google Books API** - Fetches high-quality digital book covers
- **OpenLibrary Fallback** - Alternative source if Google Books doesn't have the cover
- **Automatic Loading** - Covers appear instantly when a book is selected

### Statistics and Tracking
- **Total Spins** - Track how many times you've spun the wheel
- **Last Spun Book** - Quickly see your most recent selection
- **Current List Source** - Shows whether you're using "Full TBR", "Wheel Picks", or a "Custom" list
- **Persistent Storage** - All stats are saved locally in your browser

### Design
- **Noir Mystery Theme** - Bone, blood-red, and brass styling to match the "Who Spunnit?" banner
- **Responsive Layout** - Works on desktop and mobile devices
- **Accessibility** - High contrast and clear typography

---

## Google Sheet Structure

Your Google Sheet should have the following columns:

| Column | Content | Required | Notes |
|--------|---------|----------|-------|
| **A** | Book Title | Yes | The main book list |
| **E** | Checkbox | Optional | Check to exclude book from sync |
| **G** | Wheel Mark | Optional | Use `heathers wheel pick`, `stacys wheel pick`, or `maks wheel pick` |

### Example Sheet Setup:

| Row | A (Book Title) | ... | E (Checkbox) | ... | G (Wheel Mark) |
|-----|----------------|-----|--------------|-----|----------------|
| 1 | Book Title | ... | Skip | ... | Status |
| 2 | Throne of Glass | ... | [ ] | ... | heathers wheel pick |
| 3 | Quicksilver | ... | [x] | ... | stacys wheel pick |
| 4 | The Poison Daughter | ... | [ ] | ... | heathers wheel pick |

### How It Works:
- **Sync Full TBR** - Loads ALL unchecked books from Column A
- **Sync Wheel Picks** - Loads only unchecked books with specific values in Column G
- **Column E Checkbox** - Check = Book is ignored, Uncheck = Book is included
- **Column G Wheel Mark** - Also decides whose color a book gets on the wheel (in both sync modes)

### Tagging Picks by Hand
Each line in the book list can end with `| Name` to say who picked it, e.g. `Throne of Glass | Heather`. Synced lists are written this way automatically. Slice colors live in `PICKER_COLORS` at the top of the `<script>` in `index.html`; anyone not listed there gets a color automatically.

---

## Getting Started

### Prerequisites

1. **Google Account** - For creating and accessing Google Sheets
2. **Google Apps Script** - Built into Google Sheets
3. **Web Browser** - Modern browser with JavaScript enabled

### Step 1: Set Up Your Google Sheet

1. Create a new Google Sheet
2. Set up columns A, E, and G as described above
3. Add your book titles in Column A
4. (Optional) Add wheel marks in Column G
5. (Optional) Check boxes in Column E to skip books

### Step 2: Deploy the Google Apps Script

1. In your Google Sheet, go to **Extensions - Apps Script**
2. Delete any existing code and paste the following:

```javascript
// ===== GOOGLE APPS SCRIPT FOR BOOKCLUB WHEEL =====
// Copy this entire code into your Apps Script editor

// Column G value -> name shown on the wheel
const PICKERS = {
  'heathers wheel pick': 'Heather',
  'stacys wheel pick': 'Stacy',
  'maks wheel pick': 'Mak'
};

function doGet(e) {
  if (!e || !e.parameter) {
    return ContentService.createTextOutput(JSON.stringify({
      error: 'No parameters provided'
    })).setMimeType(ContentService.MimeType.JSON);
  }
  
  let sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('TBR List');
  if (!sheet) {
    sheet = SpreadsheetApp.getActiveSheet();
  }
  
  const action = e.parameter.action;
  const type = e.parameter.type || 'full';
  
  if (action === 'getBooks') {
    try {
      const data = sheet.getRange("A:G").getValues();
      let books = [];
      
      for (let i = 1; i < data.length; i++) {
        const title = data[i][0];
        const checked = data[i][4];
        const mark = data[i][6];
        if (checked || !title || title.toString().trim() === "") continue;

        const picker = mark ? (PICKERS[mark.toString().trim().toLowerCase()] || '') : '';
        if (type === 'wheel' && !picker) continue;

        books.push({ title: title.toString().trim(), picker: picker });
      }
      
      return ContentService.createTextOutput(JSON.stringify({
        success: true,
        books: books,
        totalBooks: books.length,
        type: type
      })).setMimeType(ContentService.MimeType.JSON);
      
    } catch (error) {
      return ContentService.createTextOutput(JSON.stringify({
        success: false,
        error: error.toString()
      })).setMimeType(ContentService.MimeType.JSON);
    }
  }
  
  return ContentService.createTextOutput(JSON.stringify({
    success: false,
    error: 'Invalid action'
  })).setMimeType(ContentService.MimeType.JSON);
}
```

3. Click **Deploy - New Deployment**
4. Click the gear icon - **Web app**
5. Set:
   - **Execute as:** `Me`
   - **Who has access:** `Anyone`
6. Click **Deploy**
7. Copy the Web App URL (e.g., `https://script.google.com/macros/s/XXXXX/exec`)

### Step 3: Set Up the HTML File

1. Copy the complete HTML code from this repository
2. Open the file in a text editor
3. Find and replace the following:
   ```javascript
   const GOOGLE_SHEETS_URL = 'YOUR_GOOGLE_APPS_SCRIPT_URL_HERE';
   ```
   Replace with your actual Web App URL

4. Find and replace the banner link:
   ```html
   <a href="YOUR_GOOGLE_SHEET_URL_HERE" target="_blank">Open Google Sheet</a>
   ```
   Replace with your actual Google Sheet URL

### Step 4: Run the Application

1. Save the HTML file
2. Open it in any modern web browser
3. Click **Sync Full TBR** or **Sync Wheel Picks** to load your books
4. Click and hold **Spin the Wheel** to charge, then release to spin!

---

## How to Use

### Syncing Books
1. Click **Sync Full TBR** to load your entire book list
2. Click **Sync Wheel Picks** to load only your curated wheel picks
3. The status will update to show which list is loaded

### Manual Editing
1. Type directly in the **Book List** textarea
2. The status will automatically change to **"Custom"**
3. Changes are saved automatically in your browser

### Spinning the Wheel
1. Click and hold the **Spin the Wheel** button
2. Hold longer for more spins (charge up)
3. Release to spin
4. Watch the wheel land on a book
5. See the book cover appear automatically
6. Enjoy the confetti celebration
- Clean black and white design

---

**Made with love by Robert Cruz**

Happy reading
