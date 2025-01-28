# University Examination Admit Card System

A web-based system for generating examination admit cards with automatic dues verification and A4-formatted printing capabilities.

## Features

- 🎓 Student authentication via roll number
- 📝 Automatic dues verification
- 🖨️ A4-formatted admit card generation
- 📅 Dynamic examination schedule display
- 🖼️ Student photo integration from Google Drive
- ✍️ Digital signature support
- 📱 Responsive design
- 🖨️ Print-optimized layout

## Prerequisites

- Google Apps Script enabled Google Sheet
- Web server to host the HTML file
- Student data in the specified format
- Controller's signature image (`s-e.png`)

## Setup

### 1. Google Sheet Setup

Create a Google Sheet with two sheets:
![Screenshot 2025-01-28 153315](https://github.com/user-attachments/assets/35b6175a-32c5-42fb-a284-8c47f1a3ccc7)

#### Sheet 1: "Student"

| Name | Course | Roll No | Date of Birth | Semester | Profile picture | Dues Status   |
| ---- | ------ | ------- | ------------- | -------- | --------------- | --------------- |

#### Sheet 2: "DateSheet"

| Date | Subject | Course | Shift | Time | Center |
| ---- | ------- | ------ | ----- | ---- | ------ |

### 2. Google Apps Script Setup

1. Open your Google Sheet
2. Go to Extensions > Apps Script
3. Copy the provided Apps Script code into the editor
   ```
   function doGet(e) {
     const output = ContentService.createTextOutput();
     output.setMimeType(ContentService.MimeType.JSON);
     
     try {
       // Basic parameter validation
       if (!e?.parameter?.rollno) {
         return output.setContent(JSON.stringify({
           error: "Please provide a valid Roll Number"
         }));
       }

       const rollNo = e.parameter.rollno.toString();
       const ss = SpreadsheetApp.getActiveSpreadsheet();
       const studentSheet = ss.getSheetByName("Student");
       const dateSheet = ss.getSheetByName("DateSheet");

       if (!studentSheet || !dateSheet) {
         return output.setContent(JSON.stringify({
           error: "Required sheets not found"
         }));
       }

       // Get student data first
       const studentData = findStudent(studentSheet, rollNo);
       if (!studentData) {
         return output.setContent(JSON.stringify({
           error: "Student not found. Please check the Roll Number."
         }));
       }

       // Get exam schedule for the student's course
       const examSchedule = findExams(dateSheet, studentData.course);
     
       const response = {
         student: studentData,
         examSchedule: examSchedule,
         exam: {
           center: "Campus"
         }
       };

       return output.setContent(JSON.stringify(response));

     } catch (error) {
       Logger.log('Error in doGet: ' + error.toString());
       return output.setContent(JSON.stringify({
         error: "An unexpected error occurred. Please try again later."
       }));
     }
   }

   function findStudent(sheet, rollNo) {
     try {
       const data = sheet.getDataRange().getValues();
       const headers = data[0];

       // Find column indices
       const rollNoCol = 2;  // Column C (0-based index)
       const nameCol = 0;    // Column A
       const courseCol = 1;  // Column B
       const dobCol = 3;     // Column D
       const semCol = 4;     // Column E
       const photoCol = 5;   // Column F
       const duesCol = 6;    // Column G

       // Search for student
       for (let i = 1; i < data.length; i++) {
         if (data[i][rollNoCol].toString() === rollNo) {
           return {
             name: data[i][nameCol],
             course: data[i][courseCol],
             rollNo: data[i][rollNoCol].toString(),
             dob: formatDate(data[i][dobCol]),
             semester: data[i][semCol],
             photo: data[i][photoCol] || null,
             duesStatus: data[i][duesCol]
           };
         }
       }
       return null;

     } catch (error) {
       Logger.log('Error in findStudent: ' + error.toString());
       throw error;
     }
   }

   function findExams(sheet, course) {
     try {
       const data = sheet.getDataRange().getValues();
       const exams = [];

       // Skip header row
       for (let i = 1; i < data.length; i++) {
         if (data[i][2] === course) {  // Column C contains course
           exams.push({
             date: formatDate(data[i][0]),      // Column A
             subject: data[i][1],               // Column B
             course: data[i][2],                // Column C
             shift: data[i][3],                 // Column D
             time: data[i][4],                  // Column E
             center: data[i][5] || "Campus"     // Column F
           });
         }
       }
     
       return exams;

     } catch (error) {
       Logger.log('Error in findExams: ' + error.toString());
       throw error;
     }
   }

   function formatDate(date) {
     if (!date) return null;
     
     try {
       if (typeof date === 'string') {
         const [day, month, year] = date.split('-');
         date = new Date(year, month - 1, day);
       }
     
       const formattedDate = Utilities.formatDate(date, Session.getScriptTimeZone(), "dd-MM-yyyy");
       return formattedDate;
     } catch (error) {
       Logger.log('Error formatting date: ' + error.toString());
       return date.toString();
     }
   }

   // Test function - run this to check if everything works
   function testScript() {
     const testEvent = {
       parameter: {
         rollno: "101"  // Test with an existing roll number
       }
     };
     
     const result = doGet(testEvent);
     Logger.log(result.getContent());
   }
   ```
4. Deploy as a web app:
   - Execute as: `Me`
   - Who has access: `Anyone`
5. Copy the deployment URL

### 3. Frontend Setup

1. Download the `index.html` file
2. Replace the `SCRIPT_URL` constant with your Apps Script deployment URL:

```javascript
const SCRIPT_URL = 'YOUR_DEPLOYMENT_URL';
```

3. Place the controller's signature image (`s-e.png`) in the same directory
4. Host the files on a web server

## Usage

1. Access the hosted webpage
2. Enter a valid roll number
3. System will verify:
4. Student existence
5. Dues status (No pending dues)
6. If verified, an A4-formatted admit card will be generated with:
7. Student details
8. Photo from Google Drive
9. Examination schedule
10. Signature spaces
11. Print button

## Important Notes

### Dues Verification

- Admit cards will ONLY generate if `Dues Status` is "No"
- Students with "Yes" in `Dues Status` will receive an error message

### Photo Requirements

- Student photos should be hosted on Google Drive
- URLs should be publicly accessible
- Recommended dimensions: 150px × 180px

### Printing

- The admit card is optimized for A4 paper size
- Use the print button in the generated card
- Print settings should be set to:
- Paper size: A4
- Margins: None
- Scale: 100%
<img width="1120" alt="image" src="https://github.com/user-attachments/assets/304d1461-4bc0-420e-ae4f-f485fa00a947" />


## File Structure

```plaintext
admit-card-system/
├── index.html          # Main application file
├── s-e.png            # Controller's signature
└── README.md          # Documentation
```

## Troubleshooting

### Common Issues

1. **Admit Card Not Generating**
2. Verify roll number exists in the sheet
3. Check dues status
4. Ensure Google Sheet permissions are correct
5. **Photo Not Displaying**
6. Verify Google Drive link is publicly accessible
7. Check if the URL is correctly formatted
8. **Print Layout Issues**
9. Ensure browser print settings are set to A4
10. Disable headers and footers in print settings
11. Set margins to none

## Security Considerations

- The system uses roll numbers for authentication
- Google Sheet access is controlled via Apps Script permissions
- Student data should be properly secured in the Google Sheet
- Consider implementing additional authentication if needed

## Contributing

To contribute to this project:

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## Support

For support, please:

1. Check the troubleshooting section
2. Review Google Apps Script documentation
3. Contact your system administrator

## Acknowledgments

- Google Apps Script platform
- Tailwind CSS for styling
- Your University/Institution name

##### This README provides comprehensive documentation for setting up and using the Admit Card System. It includes all necessary information for both administrators and developers to understand and maintain the system.

Make sure to customize the following sections based on your specific implementation:

1. Replace "YOUR_DEPLOYMENT_URL" with actual URL
2. Add any specific institutional requirements
3. Update security and support sections as needed
4. Add your institution's specific license requirements
