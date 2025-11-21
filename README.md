VAPI Appointment Booking Workflow
An n8n workflow that handles AI-powered appointment bookings through VAPI voice calls, integrating seamlessly with GoHighLevel CRM.
Overview
This workflow receives appointment booking requests from VAPI AI voice calls and automatically:

Extracts customer information from the voice call
Searches for existing contacts in GoHighLevel
Creates new contacts if needed
Books appointments in the GoHighLevel calendar
Returns success/error responses to VAPI

Workflow Architecture
VAPI Call → Webhook → Extract Data → Search Contact → Check Exists?
                                                          ├→ Yes: Use Existing
                                                          └→ No: Create New
                                                                    ↓
                                                          Book Appointment
                                                                    ↓
                                                          Success/Error Response
Prerequisites

n8n instance (self-hosted or cloud)
VAPI account with voice assistant configured
GoHighLevel account with API access
GoHighLevel API credentials

Configuration
1. GoHighLevel Setup
Location ID: ikZxyytcYlniKnh1tbwh
Calendar ID: x4HMpTaDe7tp8WrQWRr6
You'll need to update these values with your own GoHighLevel IDs:

Location ID: Found in your GHL settings
Calendar ID: Found in your calendar settings

2. n8n Credentials
Create a new HTTP Header Auth credential named "GHL E&E Landscaping API":

Header Name: Authorization
Header Value: Bearer YOUR_GHL_API_KEY

3. Webhook Configuration
The workflow uses a POST webhook endpoint. After importing, you'll receive a unique webhook URL like:
https://your-n8n-instance.com/webhook/9f1eb040-8388-463b-af4f-5ca49c31fa2e
Configure this URL in your VAPI assistant's function settings.
VAPI Function Configuration
Your VAPI assistant should call this webhook with a function that includes the following parameters:
json{
  "name": "bookAppointment",
  "description": "Books an appointment for the customer",
  "parameters": {
    "type": "object",
    "properties": {
      "customerName": {
        "type": "string",
        "description": "Full name of the customer"
      },
      "customerPhone": {
        "type": "string",
        "description": "Phone number of the customer"
      },
      "appointmentDateTime": {
        "type": "string",
        "description": "ISO 8601 datetime for the appointment (e.g., 2024-03-15T14:30:00Z)"
      }
    },
    "required": ["customerName", "customerPhone", "appointmentDateTime"]
  }
}
Workflow Nodes
1. Webhook
Receives POST requests from VAPI with appointment data.
2. Extract VAPI Data
Parses the VAPI function call and extracts:

Customer name
Customer phone
Appointment date/time

3. Search Contact
Queries GoHighLevel to find existing contacts by phone number.
4. Check if Contact Exists
Conditional logic to determine next steps based on contact existence.
5a. Use Existing Contact
If contact exists, retrieves the contact ID for appointment booking.
5b. Create New Contact
If contact doesn't exist, creates a new contact in GoHighLevel with:

Name
Phone
Tag: "ai_booked"

6. Store New Contact ID
Captures the newly created contact ID for appointment booking.
7. Book Appointment
Creates the appointment in GoHighLevel calendar using:

Contact ID
Calendar ID
Location ID
Start time (from VAPI)

8. Success Response
Returns HTTP 200 with appointment confirmation:
json{
  "status": "success",
  "message": "Appointment booked successfully",
  "appointment_time": "2024-03-15T14:30:00Z"
}
9. Error Response
Returns HTTP 400 if booking fails:
json{
  "status": "error",
  "message": "Failed to book appointment"
}
Installation

Import the workflow JSON into your n8n instance
Update the GoHighLevel credentials
Replace Location ID and Calendar ID with your values
Activate the workflow
Copy the webhook URL
Configure the webhook URL in your VAPI assistant

API Endpoints Used
GoHighLevel API (Version 2021-07-28)

GET /contacts - Search for existing contacts
POST /contacts/upsert - Create or update contacts
POST /calendars/events/appointments - Book appointments (Version 2021-04-15)

Error Handling
The workflow includes error handling for the appointment booking step:

If booking succeeds → Success Response
If booking fails → Error Response (continues execution)

Tags Applied
When creating new contacts, the workflow automatically applies the tag:

ai_booked - Identifies contacts created through AI voice booking

Testing
To test the workflow:

Manual Test via n8n:

Click the "Execute Workflow" button
Send a test webhook with sample data


Test Payload Example:

json{
  "message": {
    "toolCalls": [
      {
        "function": {
          "arguments": {
            "customerName": "John Doe",
            "customerPhone": "+15551234567",
            "appointmentDateTime": "2024-03-15T14:30:00Z"
          }
        }
      }
    ]
  }
}

Live Test with VAPI:

Make a call to your VAPI assistant
Request to book an appointment
Verify the appointment appears in GoHighLevel



Troubleshooting
Common Issues
Issue: Contact not found but should exist
Solution: Verify the phone number format matches what's in GoHighLevel
Issue: Appointment booking fails
Solution: Check that the calendar ID exists and is active
Issue: No response from workflow
Solution: Verify webhook URL is correctly configured in VAPI
Issue: 401 Authentication Error
Solution: Verify your GoHighLevel API key is valid and has correct permissions
Related Workflows
This workflow is designed to work alongside:

Outbound Caller Workflow - Initiates outbound calls through VAPI
(Link to second repository once available)

License
MIT License - Feel free to modify and use for your projects.
Support
For issues or questions:

Check the troubleshooting section above
Review n8n execution logs for detailed error messages
Verify all API credentials are correctly configured

Version History

v1.0 - Initial release with basic appointment booking functionality


Note: Remember to keep your API credentials secure and never commit them to version control.
