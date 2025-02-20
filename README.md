# Compliance Checker Web Application

## Overview
This web application processes compliance data by uploading CSV files, checking compliance through an API, and displaying results with alerts and screenshots. It includes features for data visualization, error handling, and result exports.

## Features
- CSV file upload (drag & drop or click to upload)
- Real-time processing status
- Display of compliance check results
- Screenshot viewing capability
- CSV export of results
- Error handling and display

## Input Data Format
The application expects a CSV file with the following columns:
```
PackageName, ISIN, DomicileInfo, OutstandingShares, PriorHoldingPercentage, Holdingpercentage, Alert_Id
```

Example CSV content:
```csv
PackageName,ISIN,DomicileInfo,OutstandingShares,PriorHoldingPercentage,Holdingpercentage,Alert_Id
DE,DE0005190003,DE,688720000,4.8,5.2,234178
FR,FR001400J770,FR,450000000,3.2,3.8,234179
```

## How It Works

### 1. File Upload
The application supports two methods of file upload:
- Drag and drop a CSV file into the upload area
- Click the upload area to select a file

Code example for file handling:
```javascript
function handleFile(file) {
    const reader = new FileReader();
    reader.onload = (e) => {
        // Parse CSV content
        const text = e.target.result;
        const rows = text.split('\n');
        const headers = rows[0].split(',');
        
        // Process each row
        csvData = rows.slice(1)
            .map(row => {
                const values = row.split(',');
                const obj = {};
                headers.forEach((header, i) => {
                    obj[header.trim()] = values[i]?.trim();
                });
                return obj;
            })
            .filter(row => row.ISIN);
    };
    reader.readAsText(file);
}
```

### 2. Data Processing
Once the file is uploaded, the application:
1. Validates the CSV format
2. Displays the data in a table
3. Enables the "Process Data" button

When processing starts:
```javascript
processBtn.onclick = async () => {
    for (let i = 0; i < csvData.length; i++) {
        const row = csvData[i];
        // Process each row
        const result = await processRow(row);
        // Display results
        await displayResult(row, result);
        // Wait before processing next row
        await new Promise(r => setTimeout(r, 10000));
    }
};
```

### 3. API Integration
The application makes two types of API calls:
1. Execute call - starts the compliance check
2. Status check - monitors the processing progress

Example API call:
```javascript
async function processRow(row) {
    const execResponse = await fetch('https://cloud.integrail.ai/api/d9AimxK3NrfGxWWFJ/agent/CxroQo2DwZGYnFg5e/execute', {
        method: 'POST',
        headers: {
            'Authorization': `Bearer ${API_TOKEN}`,
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({ inputs: row })
    });
}
```

### 4. Result Display
Results are displayed in three parts:
1. Header information (Alert ID, ISIN, Execution ID)
2. Report content (compliance check results)
3. Screenshots (if available)

Example result structure:
```javascript
const resultDisplay = {
    header: {
        alert_id: "234178",
        isin: "DE0005190003",
        execution_id: "dec0caf3-b753-4018-97cb-21127569331f"
    },
    report: {
        type: "Alert Report",
        flag: "True Positive",
        alert_type: "Data Quality Issue",
        justification: "Market analysis details...",
        action: "Required action details",
        timestamp: "2025-02-19T15:34:13Z"
    },
    screenshots: [
        {
            type: "image",
            url: "https://storage-service.integrail.ai/..."
        }
    ]
}
```

### 5. Export Functionality
The export function creates a CSV file containing:
- Alert ID
- ISIN
- Execution ID
- Report details (Type, Flag, Alert Type, etc.)

Example export code:
```javascript
downloadBtn.onclick = () => {
    const rows = [
        ['ISIN', 'Alert ID', 'Execution ID', 'Type', 'Flag', 'Alert Type', 'Justification', 'Action', 'Timestamp', 'Status']
    ];
    // Add data rows
    processedResults.forEach(result => {
        // Process and add each result to rows
    });
    // Create and download CSV
};
```

## Error Handling
The application handles various types of errors:

1. File Upload Errors:
```javascript
reader.onerror = (error) => {
    console.error('Error reading file:', error);
    alert('Error reading file. Please try again.');
};
```

2. Processing Errors:
```javascript
try {
    const result = await processRow(row);
} catch (error) {
    console.error('Error processing row:', error);
    // Display error in UI
}
```

3. API Errors:
```javascript
window.addEventListener('unhandledrejection', event => {
    if (event.reason.message.includes('401')) {
        alert('Session expired. Please refresh the page and try again.');
    }
});
```

## Response Format
The API returns data in this structure:
```json
{
    "status": "ok",
    "execution": {
        "_id": "dec0caf3-b753-4018-97cb-21127569331f",
        "status": "finished",
        "outputs": {
            "Alert_Id": "234178",
            "25-output": "Type: Alert Report\nFlag: True Positive...",
            "24-outputs-DEWebSharesSS": {
                "type": "image",
                "url": "https://..."
            }
        }
    }
}
```

## Usage Instructions

1. Prepare your CSV file with the required columns
2. Upload the file using drag & drop or click
3. Review the displayed data table
4. Click "Process Data" to start compliance checks
5. Wait for processing to complete
6. Review results for each row
7. Download complete results using "Download Results"

## Important Notes
- Processing each row takes time (approximately 1-2 minutes)
- There's a 10-second delay between processing rows
- The API token needs to be valid for processing to work
- Screenshots are opened in a new tab when clicked
- Export includes all processed data in CSV format

## Troubleshooting

Common issues and solutions:

1. File Upload Issues
   - Ensure file is in CSV format
   - Check CSV has correct headers
   - Verify file is not corrupted

2. Processing Issues
   - Check internet connection
   - Verify API token is valid
   - Look for console errors
   - Check CSV data format

3. Display Issues
   - Clear browser cache
   - Refresh the page
   - Check console for errors

## Technical Requirements
- Modern web browser (Chrome, Firefox, Safari, Edge)
- Internet connection
- Valid API token
- Properly formatted CSV file

For any technical issues or questions, please refer to the console logs for detailed error messages.
