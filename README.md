# moodle-gptzero-demo
- Start Date: 2024-04-14
- RFC PR:
- Issue:

# Summary

We propose to implement a Moodle plugin that will integrate the GPTZero API. This plugin is intended to detecting AI-generated content in student assignments. It will be designed as part of Moodle's plagiarism component type. Teachers will be able to view the likelihood of whether an assignment is being AI-generated, human-written, or a mix along with a further breakdown of the text.

# Motivation

With the increasing use of generative AI in education, it is essential to have tools that can distinguish between human and AI-generated content. This plugin seeks to preserve what's human and bring transparency to humans navigating a world filled with AI content.
- Enhance academic integrity by identifying AI-generated content.
- Give educators access to GPTZero, the gold standard in AI detection.
- Offer a non-invasive integration option for educational institutions using Moodle.

# Detailed design

Plugin Architecture
- Plugin Component Type: Plagiarism Plugin
- File handling: Utilize Moodle's file_storage API to upload and save documents
- Trigger: Use Moodle's event system to trigger our API call when an assignment is submitted. Our hook will listen to the assignsubmission_file_uploaded event. When this event occurs, we will make the API call.
```php
// Example of hooking into an event
$eventManager->addListener('assignsubmission_file_uploaded', 'processSubmission');
function processSubmission($eventData) {
    $assignmentContent = $eventData->getContent();
    performGPTZeroScan($assignmentContent);
}
```
- Endpoint and Request Type: The plugin will use the cURL library to make POST requests to the GPTZero API endpoint (https://api.gptzero.me/v2/predict/files). This endpoint is designed to receive files for AI content analysis.
- API Key: The x-api-key header will be included to authenticate the request. This key should securely configured by the site admin and securely stored in lang/en/plagiarism_gptzero.php.
- Database Schema: Extends Moodle's database schema to include tables for storing results of the AI detection scans. Includes columns for scan_id, assignment_id, user_id, predicted_class, confidence_score, and a JSON column for storing the detailed analysis results.

UI
- Gradebook Quickview: For the teacher role, users can see a table overview of all students and their submission for the assignment due to the built-in Moodle component. The plugin will integrate into this table and add a column that shows the predicted class (i.e. ai-generated, mixed, human) and its confidence category (high, medium, low). There will also be an option to view a deepscan.
- Deep Scan Interface: A detailed page that shows the text from the document and highlights sentences predicted as AI-written. Classification (i.e. "We are highly confident this text was ai generated") and a probability breakdown will also be displayed.

Compliance
- Students must agree to a disclosure statement.
- Abide by GDBR

Unit Testing with PHPUnit:
- API Request Function
- Response Handling
- Database Interaction

Behavioural Testing with Behat:
- Assignment Submission Workflow
- Error Handling in UI
- Access Control (only teacher role)

# Alternatives

Direct Moodle Core Modifications: Although potentially more powerful, modifying Moodle's core could lead to compatibility and maintenance issues. Additionally, it makes it much harder to be integrated into existing websites working on their own forks of Moodle.
Use of Other Plagiarism Detection Tools: While other tools exist, they may not offer the specific AI detection capabilities provided by GPTZero.

# Adoption strategy

This plugin will be available as a plugin for Moodle (ideally accessible from Moodle's plugin directory)

# How we teach this

Documentation will include:
- Installation and setup guides.
- Examples of how to interpret the plugin's outputs.
- Best practices for integrating AI content detection in academic settings.
