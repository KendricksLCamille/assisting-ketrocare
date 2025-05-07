I chose to use AI since this is so simple it didn't really make sense wasting my time refreshing myself when I knew exactly what I wanted to build. Sadly, the key-value input was a little complicated, so I had to fix that myself, as well as making sure the output section actually looked proper, but it handled much of the busy work.

**Website:** [deepai.org/chat](https://deepai.org/chat)

**Prompt:**

    Generate a single html page with javascript using bootstrap style css.\

    It is basically a form with a submit on the left half of the screen and an output which is made up of an uneditable text and a button on the bottom which copies the uneditable textbox to the user's clipboard.

    The form should have a place for entering Notes about the Patient, Admission, Medication, Follow Up, Patient Education, and Etc.

    The form also needs the ability to add additional inputs which have two forms: either a single line textbox or two text boxes with one being the key and the other being the value.

    Finally, the keys for the key-value inputs are editable drop downs, so if a case-insensitive match occurs, it should show options that are stored in web storage. If nothing is there, let the user write whatever they want, and once they click submit, add the key-value to the list if it isn't already there.

    Finally, print out that information on the output text block in a nice format.

**Output from AI Below(HTML Code):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Medical Notes Generator</title>
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        /* Custom CSS for better layout */
        .container-fluid {
            padding: 20px;
        }
        .form-section, .output-section {
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 5px;
            margin-bottom: 20px;
        }
        .form-section {
            background-color: #f8f9fa;
        }
        .output-section {
            background-color: #e9ecef;
        }
        .output-textarea {
            width: 100%;
            height: 400px;
            resize: none;
            font-family: monospace;
            border: 1px solid #ccc;
            padding: 10px;
            overflow-y: auto;
            white-space: pre-wrap; /* Preserve line breaks */
        }
        .add-additional-input {
            margin-top: 10px;
        }
        .key-value-pair {
            display: flex;
            gap: 10px;
            align-items: center;
        }
        .key-value-pair .form-control {
            flex-grow: 1;
        }
        .key-value-pair .remove-btn {
            flex-shrink: 0;
        }
        .additional-inputs-container {
            margin-top: 15px;
        }
        .form-select-key {
            width: 100%; /* Ensure dropdown takes full width of its column */
        }
    </style>
</head>
<body>

<div class="container-fluid">
    <div class="row">
        <div class="col-md-6 form-section">
            <h2>Patient Notes Form</h2>
            <form id="notesForm">
                <div class="mb-3">
                    <label for="patientNotes" class="form-label">Notes about the Patient</label>
                    <textarea class="form-control" id="patientNotes" rows="3"></textarea>
                </div>
                <div class="mb-3">
                    <label for="admissionNotes" class="form-label">Admission</label>
                    <textarea class="form-control" id="admissionNotes" rows="3"></textarea>
                </div>
                <div class="mb-3">
                    <label for="medicationNotes" class="form-label">Medication</label>
                    <textarea class="form-control" id="medicationNotes" rows="3"></textarea>
                </div>
                <div class="mb-3">
                    <label for="followUpNotes" class="form-label">Follow Up</label>
                    <textarea class="form-control" id="followUpNotes" rows="3"></textarea>
                </div>
                <div class="mb-3">
                    <label for="educationNotes" class="form-label">Patient Education</label>
                    <textarea class="form-control" id="educationNotes" rows="3"></textarea>
                </div>
                <div class="mb-3">
                    <label class="form-label">Additional Information</label>
                    <div id="additionalInputsContainer" class="additional-inputs-container">
                        <!-- Additional inputs will be added here -->
                    </div>
                    <button type="button" class="btn btn-secondary btn-sm add-additional-input" onclick="addAdditionalInput('single')">Add Single Line Input</button>
                    <button type="button" class="btn btn-secondary btn-sm add-additional-input" onclick="addAdditionalInput('keyvalue')">Add Key-Value Input</button>
                </div>

                <button type="submit" class="btn btn-primary">Generate Notes</button>
            </form>
        </div>

        <div class="col-md-6 output-section">
            <h2>Generated Notes</h2>
            <textarea class="output-textarea" id="outputNotes" readonly></textarea>
            <button class="btn btn-secondary mt-3" onclick="copyOutput()">Copy Notes</button>
        </div>
    </div>
</div>

<!-- Bootstrap JS (optional, but good for some Bootstrap components) -->
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
<script>
    const additionalInputsContainer = document.getElementById('additionalInputsContainer');
    const outputNotesTextarea = document.getElementById('outputNotes');
    const keyStorageKey = 'medicalNoteKeys';

    // Load existing keys from local storage
    let existingKeys = JSON.parse(localStorage.getItem(keyStorageKey)) || [];

    function addAdditionalInput(type) {
        const inputGroup = document.createElement('div');
        inputGroup.classList.add('mb-2', 'additional-input-group');

        if (type === 'single') {
            inputGroup.innerHTML = `
                <div class="input-group">
                    <input type="text" class="form-control" placeholder="Enter additional note...">
                    <button type="button" class="btn btn-outline-danger remove-btn" onclick="removeInput(this)">Remove</button>
                </div>
            `;
        } else if (type === 'keyvalue') {
            inputGroup.innerHTML = `
                <div class="key-value-pair">
                    <input type="text" class="form-control form-select-key" placeholder="Key" list="keyOptions" oninput="filterKeyOptions(this)">
                    <datalist id="keyOptions"></datalist>
                    <input type="text" class="form-control" placeholder="Value">
                    <button type="button" class="btn btn-outline-danger remove-btn" onclick="removeInput(this)">Remove</button>
                </div>
            `;
            updateKeyDatalist(); // Populate datalist on creation
        }

        additionalInputsContainer.appendChild(inputGroup);
    }

    function removeInput(button) {
        button.closest('.additional-input-group').remove();
    }

    function updateKeyDatalist() {
        const datalist = document.getElementById('keyOptions');
        datalist.innerHTML = ''; // Clear existing options
        existingKeys.forEach(key => {
            const option = document.createElement('option');
            option.value = key;
            datalist.appendChild(option);
        });
    }

    function filterKeyOptions(inputElement) {
        const value = inputElement.value.toLowerCase();
        const datalist = document.getElementById('keyOptions');
        datalist.innerHTML = ''; // Clear existing options

        // Filter existing keys based on the input value
        const filteredKeys = existingKeys.filter(key => key.toLowerCase().includes(value));

        // Add filtered keys to the datalist
        filteredKeys.forEach(key => {
            const option = document.createElement('option');
            option.value = key;
            datalist.appendChild(option);
        });

        // If the current input value doesn't match any existing key exactly (case-insensitive),
        // consider adding it as a potential new key (this is handled on submit)
    }

    document.getElementById('notesForm').addEventListener('submit', function(event) {
        event.preventDefault();

        const patientNotes = document.getElementById('patientNotes').value.trim();
        const admissionNotes = document.getElementById('admissionNotes').value.trim();
        const medicationNotes = document.getElementById('medicationNotes').value.trim();
        const followUpNotes = document.getElementById('followUpNotes').value.trim();
        const educationNotes = document.getElementById('educationNotes').value.trim();

        let generatedNotes = '';

        if (patientNotes) {
            generatedNotes += 'Notes about the Patient:\n' + patientNotes + '\n\n';
        }
        if (admissionNotes) {
            generatedNotes += 'Admission:\n' + admissionNotes + '\n\n';
        }
        if (medicationNotes) {
            generatedNotes += 'Medication:\n' + medicationNotes + '\n\n';
        }
        if (followUpNotes) {
            generatedNotes += 'Follow Up:\n' + followUpNotes + '\n\n';
        }
        if (educationNotes) {
            generatedNotes += 'Patient Education:\n' + educationNotes + '\n\n';
        }

        const additionalInputs = additionalInputsContainer.querySelectorAll('.additional-input-group');
        if (additionalInputs.length > 0) {
            generatedNotes += 'Additional Information:\n';
            additionalInputs.forEach(inputGroup => {
                const singleInput = inputGroup.querySelector('.input-group input[type="text"]');
                const keyValuePair = inputGroup.querySelector('.key-value-pair');

                if (singleInput) {
                    const value = singleInput.value.trim();
                    if (value) {
                        generatedNotes += `- ${value}\n`;
                    }
                } else if (keyValuePair) {
                    const keyInput = keyValuePair.querySelector('.form-control:nth-child(1)');
                    const valueInput = keyValuePair.querySelector('.form-control:nth-child(2)');
                    const key = keyInput.value.trim();
                    const value = valueInput.value.trim();

                    if (key || value) { // Only add if key or value is present
                        generatedNotes += `- ${key}: ${value}\n`;

                        // Add new key to storage if it's not already there (case-insensitive)
                        const keyExists = existingKeys.some(existingKey => existingKey.toLowerCase() === key.toLowerCase());
                        if (key && !keyExists) {
                            existingKeys.push(key);
                            localStorage.setItem(keyStorageKey, JSON.stringify(existingKeys));
                            updateKeyDatalist(); // Update datalist after adding
                        }
                    }
                }
            });
            generatedNotes += '\n';
        }

        outputNotesTextarea.value = generatedNotes;
    });

    function copyOutput() {
        outputNotesTextarea.select();
        outputNotesTextarea.setSelectionRange(0, 99999); // For mobile devices

        try {
            navigator.clipboard.writeText(outputNotesTextarea.value).then(function() {
                // Optional: Provide feedback to the user (e.g., a small tooltip or message)
                console.log('Notes copied to clipboard');
            }, function(err) {
                console.error('Could not copy text: ', err);
                // Fallback for older browsers or if clipboard API is not available
                alert('Could not copy notes automatically. Please copy the text manually from the box.');
            });
        } catch (err) {
            console.error('Could not copy text: ', err);
            // Fallback for older browsers or if clipboard API is not available
            alert('Could not copy notes automatically. Please copy the text manually from the box.');
        }
    }

    // Initial update of datalist on page load
    updateKeyDatalist();
</script>

</body>
</html>
```

*   **Explanation:**

    1.  **HTML Structure:**
        *   Uses `<!DOCTYPE html>`, `<html>`, `<head>`, and `<body>` for a standard HTML page.
        *   Includes Bootstrap CSS (`<link>`) for styling.
        *   Uses a `container-fluid` and `row` with `col-md-6` to divide the screen into two halves for the form and the output.
        *   Two main sections: `form-section` and `output-section`.
        *   The form (`<form id="notesForm">`) contains `textarea` elements for the standard notes sections.
        *   An `additionalInputsContainer` `div` will hold the dynamically added inputs.
        *   Buttons are provided to add single-line and key-value inputs.
        *   The output section contains a `textarea` (`id="outputNotes"`) with the `readonly` attribute to prevent editing and a "Copy Notes" button.

    2.  **CSS Styling (Bootstrap and Custom):**
        *   Uses Bootstrap classes like `container-fluid`, `row`, `col-md-6`, `form-label`, `form-control`, `btn`, `mb-3`, `input-group`, etc.
        *   Custom CSS is added to:
            *   Add padding and borders to the sections.
            *   Set background colors for better visual separation.
            *   Style the output textarea for better readability and overflow handling.
            *   Control the layout of the additional inputs and key-value pairs.

    3.  **JavaScript Functionality:**
        *   **`additionalInputsContainer` and `outputNotesTextarea`:** Get references to the relevant DOM elements.
        *   **`keyStorageKey`:** Defines the key used to store and retrieve the list of keys from `localStorage`.
        *   **`existingKeys`:** An array to hold the keys stored in `localStorage`. It's initialized by attempting to parse the stored data or defaulting to an empty array.
        *   **`addAdditionalInput(type)`:**
            *   Takes a `type` argument ('single' or 'keyvalue').
            *   Creates a new `div` element (`inputGroup`) with Bootstrap classes for spacing and layout.
            *   Based on the `type`:
                *   **Single Line:** Creates an `input-group` with a single `text` input and a "Remove" button.
                *   **Key-Value:** Creates a `key-value-pair` div with two `text` inputs (one for the key, one for the value) and a "Remove" button.
                *   The key input for key-value pairs has a `list="keyOptions"` attribute, linking it to a `<datalist>` element.
                *   An `oninput="filterKeyOptions(this)"` event listener is added to the key input to handle filtering of datalist options as the user types.
            *   Appends the created `inputGroup` to the `additionalInputsContainer`.
            *   Calls `updateKeyDatalist()` to ensure the datalist is populated when a new key-value input is added.
        *   **`removeInput(button)`:**
            *   Takes the clicked "Remove" button as an argument.
            *   Uses `closest('.additional-input-group')` to find the parent container of the input and removes it from the DOM.
        *   **`updateKeyDatalist()`:**
            *   Gets a reference to the `<datalist>` element (`id="keyOptions"`).
            *   Clears the current options in the datalist.
            *   Iterates through the `existingKeys` array and creates an `<option>` element for each key, setting its `value` to the key.
            *   Appends the option to the datalist.
        *   **`filterKeyOptions(inputElement)`:**
            *   Takes the key input element as an argument.
            *   Gets the current value of the input and converts it to lowercase for case-insensitive matching.
            *   Gets the datalist element.
            *   Clears the current datalist options.
            *   Filters the `existingKeys` array to find keys that include the input value (case-insensitive).
            *   Adds the filtered keys as `<option>` elements to the datalist.
            *   If the current input value doesn't match any existing key exactly (case-insensitive), consider adding it as a potential new key (this is handled on submit).
        *   **Form Submission (`notesForm.addEventListener('submit', ...)`):**
            *   Prevents the default form submission behavior.
            *   Gets the values from the standard text areas.
            *   Initializes an empty string `generatedNotes`.
            *   Appends the standard notes to `generatedNotes` if they are not empty, adding labels and newlines for formatting.
            *   Iterates through the dynamically added input groups:
                *   Checks if it's a single input or a key-value pair.
                *   Gets the values from the inputs.
                *   If the input has a value (or key/value for key-value pairs), appends it to `generatedNotes` with appropriate formatting (bullet points for additional notes).
                *   **For key-value pairs:**
                    *   Checks if the entered key already exists in `existingKeys` (case-insensitive).
                    *   If the key is not empty and does not exist, it's added to the `existingKeys` array.
                    *   The updated `existingKeys` array is saved back to `localStorage` after stringifying it.
                    *   `updateKeyDatalist()` is called to refresh the datalist options with the new key.
            *   Sets the `value` of the `outputNotesTextarea` to the `generatedNotes` string.
        *   **`copyOutput()`:**
            *   Selects the text in the `outputNotesTextarea`.
            *   Uses the `navigator.clipboard.writeText()` API to copy the selected text to the clipboard.
            *   Includes basic error handling and a fallback `alert` for browsers that don't support the Clipboard API.
        *   **Initial Datalist Update:** Calls `updateKeyDatalist()` when the page loads to populate the datalist with any existing keys from local storage.

    4.  **Web Storage (localStorage):**
        *   The `existingKeys` array is used to store the unique keys entered in the key-value pairs.
        *   `localStorage.getItem(keyStorageKey)` retrieves the stored keys (if any) when the page loads.
        *   `localStorage.setItem(keyStorageKey, JSON.stringify(existingKeys))` saves the updated list of keys back to local storage whenever a new unique key is entered during form submission.
        *   `JSON.parse()` is used to convert the string retrieved from `localStorage` back into a JavaScript array.
        *   `JSON.stringify()` is used to convert the JavaScript array into a string before storing it in `localStorage`.

*   **How to Use:**

    1.  Save the code as an HTML file (e.g., `medical_notes.html`).
    2.  Open the HTML file in your web browser.
    3.  Fill out the standard notes sections.
    4.  Use the "Add Single Line Input" and "Add Key-Value Input" buttons to add additional fields as needed.
    5.  For key-value inputs, start typing in the "Key" field. If you've used that key before (and it's stored in local storage), options will appear in a dropdown. You can select an existing key or type a new one.
    6.  Fill in the values for the additional inputs.
    7.  Click the "Generate Notes" button. The formatted notes will appear in the output box.
    8.  Click the "Copy Notes" button to copy the generated text to your clipboard.

This single HTML file contains all the necessary code (HTML, CSS, and JavaScript) to run the application directly in a web browser.
