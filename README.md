# ServiceNow ↔ Google Gemini AI Integration

Integrate Google Gemini AI with ServiceNow using an **Outbound REST Message**, a reusable **Script Include**, and a **UI Action button**. A user enters a prompt in ServiceNow, clicks **Generate Gemini Response**, and the generated response is saved to a custom table.

> Source: README created from the 11-page setup document provided with this project. The endpoint/model and API configuration may need updating to match the Gemini API version and model available to your Google Cloud project.

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [How It Works](#2-how-it-works)
3. [Prerequisites](#3-prerequisites)
4. [Create a Gemini API Key](#4-create-a-gemini-api-key)
5. [Create a System Property for the API Key](#5-create-a-system-property-for-the-api-key)
6. [Create the Outbound REST Message](#6-create-the-outbound-rest-message)
7. [Create the HTTP Method](#7-create-the-http-method)
8. [Configure Query Parameters and Headers](#8-configure-query-parameters-and-headers)
9. [Configure the Request Body](#9-configure-the-request-body)
10. [Test the REST Message](#10-test-the-rest-message)
11. [Create the Prompt History Table](#11-create-the-prompt-history-table)
12. [Create the Script Include](#12-create-the-script-include)
13. [Create the UI Action Button](#13-create-the-ui-action-button)
14. [End-to-End Testing](#14-end-to-end-testing)
15. [Troubleshooting](#15-troubleshooting)
16. [Security and Production Notes](#16-security-and-production-notes)
17. [Expected Outcome](#17-expected-outcome)

## 1. Project Overview

### Objective

The integration allows a ServiceNow user to:

- Enter a dynamic prompt.
- Click **Generate Gemini Response**.
- Send the prompt to Google Gemini through the Gemini API.
- Receive AI-generated text.
- Store the prompt, response, status, and any error in a custom ServiceNow table.

### Technologies Used

- ServiceNow platform
- ServiceNow Outbound REST Message / RESTMessageV2
- ServiceNow Script Include
- ServiceNow UI Action (server-side)
- Google Gemini API
- JavaScript

## 2. How It Works

```text
User enters prompt
        |
        v
Gemini Prompt History record
        |
        v
Generate Gemini Response UI Action
        |
        v
GeminiIntegration Script Include
        |
        v
Google_Gemini Outbound REST Message
        |
        v
Google Gemini API
        |
        v
Parse API response
        |
        v
Save response + status to ServiceNow
```

## 3. Prerequisites

Before beginning, ensure you have:

1. A ServiceNow instance with sufficient permissions to create tables, Script Includes, REST Messages, and UI Actions.
2. A Google Cloud account.
3. The Generative Language API enabled for the relevant Google project.
4. A Gemini API key.
5. Permission to test outbound HTTPS requests from the ServiceNow instance.

## 4. Create a Gemini API Key

1. Open Google AI Studio / the Google Cloud configuration used for Gemini.
2. Create or select a project.
3. Enable the Generative Language API if required.
4. Generate an API key.
5. Keep the key private. Do not commit it to GitHub or paste it into client-side code.

The source document uses API-key authentication through a query parameter named `key`; it does not use OAuth for this example.

## 5. Create a System Property for the API Key

Storing the key in a system property is preferable to hardcoding it into scripts or source control.

1. In ServiceNow, open **System Properties** (the exact module can vary by release and configuration).
2. Create a string property:
   - **Name:** `gemini.api.key`
   - **Value:** your Gemini API key
   - **Description:** API key used by the Google Gemini integration
3. Save the property.
4. Restrict access to the property according to your organization's security policy.

> Important: Do not put the literal API key in this README. The key should remain private and should not be exposed in client-side scripts, logs, screenshots, or source control.

## 6. Create the Outbound REST Message

1. Navigate to **System Web Services → Outbound → REST Message**.
2. Select **New**.
3. Configure the REST Message:

   | Field | Value |
   |---|---|
   | Name | `Google_Gemini` |
   | Endpoint | `https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent` |

4. Save the record.

> Model availability changes over time. If the model in the endpoint is no longer available to your project, replace it with a currently supported Gemini model and update the endpoint accordingly.

## 7. Create the HTTP Method

1. Open the `Google_Gemini` REST Message.
2. In the HTTP Methods related list, select **New**.
3. Configure the method:

   | Field | Value |
   |---|---|
   | Name | `generate` |
   | HTTP method | `POST` |

4. Save the HTTP method.

## 8. Configure Query Parameters and Headers

Open the `generate` HTTP Method and configure its HTTP Request settings.

### Query parameter

Add this query parameter:

| Name | Value |
|---|---|
| `key` | Your Gemini API key / configured property value |

The source document recommends using the system property `gemini.api.key`. The exact way to reference a property from a REST Message query-parameter field depends on the ServiceNow release and field configuration. Do not assume that writing `${gs.getProperty('gemini.api.key')}` in a REST Message field will be evaluated as JavaScript. Verify the supported variable-substitution method in your instance. If necessary, set the key on the REST request in server-side code using a supported API, while keeping the secret server-side.

### HTTP header

Add:

| Name | Value |
|---|---|
| `Content-Type` | `application/json` |

## 9. Configure the Request Body

Set the HTTP method's request body to the following JSON:

```json
{
  "contents": [
    {
      "parts": [
        {
          "text": "${prompt}"
        }
      ]
    }
  ]
}
```

`${prompt}` is the variable placeholder used by this example for dynamic prompt substitution.

Save the HTTP method after entering the request body.

> **Implementation note:** JSON request bodies require correct escaping. The supplied example uses `setStringParameterNoEscape('prompt', promptText)` later in the Script Include. Verify that prompts containing quotes, backslashes, or newlines are serialized as valid JSON. For robust production code, construct the request body with `JSON.stringify()` and use a REST Message configuration that accepts the serialized body, or otherwise ensure correct JSON escaping.

## 10. Test the REST Message

1. Open the `generate` HTTP Method.
2. Select **Test**.
3. Provide a test value for the variable:

   ```text
   prompt = Explain ServiceNow Incident Management in simple words
   ```

4. Run the test.

### Expected result

- HTTP status: `200`
- Response body contains Gemini-generated text, usually within the `candidates` response structure.

If the test fails, check the API key, endpoint/model availability, API enablement, query parameter, request JSON, and outbound network access.

## 11. Create the Prompt History Table

1. Navigate to **System Definition → Tables**.
2. Select **New**.
3. Create the table:

   | Field | Value |
   |---|---|
   | Label | `Gemini Prompt History` |
   | Name | `u_gemini_prompt_history` |

4. Save the table.

### Add columns

Create the following fields on the table. Field type labels can differ slightly by ServiceNow release.

| Label | Column name | Suggested type |
|---|---|---|
| Prompt | `u_prompt` | String with sufficient length / Large String |
| Gemini Response | `u_response` | Large String |
| Status | `u_status` | Choice |
| Error | `u_error` | String or Large String |

### Configure status choices

For `u_status`, add these choices:

- `Requested`
- `Success`
- `Failed`

Use the exact stored choice values consistently in scripts and choice configuration. Depending on how choices are configured, the stored value may be a separate value from the displayed label.

## 12. Create the Script Include

1. Navigate to **System Definition → Script Includes**.
2. Select **New**.
3. Configure:

   | Field | Value |
   |---|---|
   | Name | `GeminiIntegration` |
   | Client callable | Unchecked |
   | Active | Checked |
   | Application scope | Use the scope where the REST Message and table are available |

4. Add the following server-side script:

```javascript
var GeminiIntegration = Class.create();
GeminiIntegration.prototype = {
    initialize: function () {},

    getResponse: function (promptText) {
        try {
            var rm = new sn_ws.RESTMessageV2('Google_Gemini', 'generate');

            // The REST Message must have a correctly configured key parameter.
            // Keep the API key server-side; never expose it to the browser.
            rm.setStringParameterNoEscape('prompt', promptText);

            var response = rm.execute();
            var status = response.getStatusCode();
            var body = response.getBody();

            if (status !== 200) {
                return {
                    success: false,
                    error: 'Gemini API returned HTTP ' + status + ': ' + body
                };
            }

            var obj = JSON.parse(body);

            if (!obj.candidates ||
                !obj.candidates[0] ||
                !obj.candidates[0].content ||
                !obj.candidates[0].content.parts ||
                !obj.candidates[0].content.parts[0]) {
                return {
                    success: false,
                    error: 'The Gemini API response did not contain a text candidate.'
                };
            }

            var generatedText = obj.candidates[0].content.parts[0].text;

            return {
                success: true,
                text: generatedText
            };

        } catch (ex) {
            return {
                success: false,
                error: ex.message || String(ex)
            };
        }
    },

    type: 'GeminiIntegration'
};
```

5. Save the Script Include.

### Important configuration notes

- The code expects a REST Message named `Google_Gemini` and an HTTP method named `generate`.
- The REST Message must correctly resolve both the `prompt` variable and the API key query parameter.
- `setStringParameterNoEscape()` does not automatically make arbitrary inserted text safe for JSON. Test special characters and use robust JSON serialization for production.
- If the REST Message is in another application scope, configure the appropriate application access and use the correct REST Message name/reference for your instance.

## 13. Create the UI Action Button

1. Navigate to **System Definition → UI Actions**.
2. Select **New**.
3. Configure the UI Action:

   | Field | Value |
   |---|---|
   | Name | `Generate Gemini Response` |
   | Table | `u_gemini_prompt_history` |
   | Action name | `generate_gemini` |
   | Form button | Checked |
   | Client | Unchecked |
   | Show update | Checked |
   | Active | Checked |

4. If you want to hide the button after a successful response, use this condition:

   ```javascript
   current.u_status != 'Success'
   ```

5. Add the following **server-side** UI Action script:

```javascript
if (!current.u_prompt) {
    gs.addErrorMessage('Please enter a prompt first.');
    action.setRedirectURL(current);
} else {
    current.u_status = 'Requested';
    current.u_error = '';
    current.u_response = '';
    current.update();

    var gemini = new GeminiIntegration();
    var result = gemini.getResponse(current.u_prompt.toString());

    if (result.success) {
        current.u_response = result.text;
        current.u_status = 'Success';
        current.u_error = '';
        gs.addInfoMessage('Gemini response generated successfully.');
    } else {
        current.u_status = 'Failed';
        current.u_error = result.error;
        gs.addErrorMessage('Failed to generate Gemini response. Check the error field and system logs.');
    }

    current.update();
    action.setRedirectURL(current);
}
```

6. Save the UI Action.

### What this script does

1. Checks that a prompt has been entered.
2. Sets the status to `Requested`.
3. Calls the `GeminiIntegration` Script Include.
4. Saves the generated response and sets the status to `Success`, or saves the error and sets the status to `Failed`.
5. Redirects the user back to the current record.

> The sample follows the source document's server-side UI Action flow. Validate behavior in a development instance first, especially the two record updates and the way your instance handles unsaved form values.

## 14. End-to-End Testing

Follow these steps in order:

1. Open the **Gemini Prompt History** table/list.
2. Click **New**.
3. Enter a prompt in the Prompt field, for example:

   ```text
   Explain Incident, Problem, and Change Management in ServiceNow using simple examples.
   ```

4. Save the record first.
5. Open the saved record and click **Generate Gemini Response**.
6. Wait for the server-side REST request to finish.
7. Review the fields:

   - **Prompt:** your original prompt
   - **Gemini Response:** generated answer, if successful
   - **Status:** `Success` or `Failed`
   - **Error:** error details, if a failure occurred

### Acceptance checklist

- [ ] REST Message test succeeds.
- [ ] A new Gemini Prompt History record can be created.
- [ ] The Generate Gemini Response button appears on the form.
- [ ] A valid prompt reaches Gemini.
- [ ] The response is stored in `u_response`.
- [ ] Status changes to `Success` after a successful response.
- [ ] A failure sets status to `Failed` and records an error.
- [ ] The API key is not exposed to client-side code or source control.

## 15. Troubleshooting

| Symptom | What to check |
|---|---|
| HTTP `400` | Check the request JSON, prompt substitution, endpoint, and model name. |
| HTTP `401` or `403` | Check the API key, API enablement, project access, and key restrictions. |
| HTTP `404` | Confirm the endpoint and Gemini model are available. |
| HTTP `429` | Check quota, rate limits, and billing/project restrictions if applicable. |
| Empty response / missing `candidates` | Inspect the full response body; the API may return a block, safety response, or a different error structure. |
| Prompt placeholder is not replaced | Verify the variable substitution configuration for the REST Message and HTTP method. |
| JSON parsing error | Inspect the outgoing body and make sure prompts with quotes, newlines, and backslashes are escaped correctly. |
| UI Action does not appear | Check the table, active flag, Form button setting, condition, and user permissions. |
| Script Include cannot be found | Check the application scope, active flag, and cross-scope access. |
| Response is not saved | Check field names, field types, ACLs, server logs, and whether the UI Action reaches its final update. |

## 16. Security and Production Notes

Before using this pattern in production:

1. **Protect the API key.** Keep it in a protected server-side configuration and restrict access.
2. **Never commit secrets.** Do not place API keys in GitHub, README files, client scripts, or screenshots.
3. **Validate prompts.** Apply suitable input limits and access controls.
4. **Handle errors carefully.** Avoid exposing sensitive request/response details to users; log only what your organization's policy allows.
5. **Review data retention.** Prompts and AI responses may contain confidential information. Define access and retention rules for the history table.
6. **Check permissions.** Configure table ACLs so only authorized users can read and create prompt history records.
7. **Validate model/API behavior.** Gemini model names, availability, response formats, quotas, and API requirements can change.
8. **Test in development first.** Validate API key handling, variable substitution, JSON escaping, timeouts, error paths, and cross-scope access before deploying.
9. **Treat generated content as untrusted.** Review AI output before using it for operational or customer-facing decisions.

## 17. Expected Outcome

After configuration, a user can enter a prompt in ServiceNow and click **Generate Gemini Response**. ServiceNow sends the prompt to Google Gemini through an Outbound REST Message, parses the result in a reusable Script Include, and stores the response and status in the **Gemini Prompt History** table.

### Main components created

- Outbound REST Message: `Google_Gemini`
- HTTP Method: `generate`
- System Property: `gemini.api.key`
- Custom Table: `u_gemini_prompt_history`
- Script Include: `GeminiIntegration`
- UI Action: `Generate Gemini Response`

---

**Implementation source:** The supplied document, *ServiceNow ↔ Google Gemini Integration Using Outbound REST Message + UI Action Button* (11 pages). This README preserves its overall workflow and configuration while adding implementation notes where configuration can vary by ServiceNow release or Gemini API availability.
