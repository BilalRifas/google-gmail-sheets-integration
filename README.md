# Ballerina google-gmail-spreadsheet-integration
Application for event notification for registered members using Ballerina

Google Sheets is an online spreadsheet that lets users create and format spreadsheets and simultaneously work with other people. The Google Spreadsheet endpoint allows you to access the Google Spreadsheet API Version v4 through Ballerina.

# Compatibility
Ballerina Language Version 	Google Spreadsheet API Version
0.975.0 	V4

# Working with GSheets Endpoint actions

All the actions return valid response or SpreadsheetError. If the action is a success, then the requested resource will be returned. Else SpreadsheetError object will be returned.

In order for you to use the GSheets Endpoint, first you need to create a GSheets Client endpoint.

import wso2/gsheets4;

endpoint gsheets4:Client spreadsheetClientEP {
    clientConfig:{
        auth:{
            accessToken:"<your_accessToken>",
            refreshToken:"<your_refreshToken>",
            clientId:"<your_clientId>",
            clientSecret:"<your_clientSecret>"
        }
    }
};

Then the endpoint actions can be invoked as var response = spreadsheetClientEP -> actionName(arguments).
Sample

import ballerina/config;
import ballerina/io;
import wso2/gsheets4;

function main(string... args) {
    endpoint gsheets4:Client spreadsheetClientEP {
        clientConfig:{
            auth:{
                accessToken:config:getAsString("ACCESS_TOKEN"),
                refreshToken:config:getAsString("REFRESH_TOKEN"),
                clientId:config:getAsString("CLIENT_ID"),
                clientSecret:config:getAsString("CLIENT_SECRET")
            }
        }
    };

    gsheets4:Spreadsheet spreadsheet = new;
    var response = spreadsheetClientEP->openSpreadsheetById("abc1234567");
    match response {
        gsheets4:Spreadsheet spreadsheetRes => {
            spreadsheet = spreadsheetRes;
        }
        gsheets4:SpreadsheetError err => {
            io:println(err);
        }
    }

    io:println(spreadsheet);
}



# Connects to Gmail from Ballerina.


You can now enter the credentials in the HTTP client config.

endpoint gmail:Client gmailEP {
    clientConfig:{
        auth:{
            accessToken:accessToken,
            clientId:clientId,
            clientSecret:clientSecret,
            refreshToken:refreshToken
        }
    }
};

The sendMessage function sends an email. MessageRequest is a structure that contains all the data that is required to send an email. The userId represents the authenticated user and can be a Gmail address or ‘me’ (the currently authenticated user).

string userId = "me";
gmail:MessageRequest messageRequest;
messageRequest.recipient = "recipient@mail.com";
messageRequest.sender = "sender@mail.com";
messageRequest.cc = "cc@mail.com";
messageRequest.subject = "Email-Subject";
messageRequest.messageBody = "Email Message Body Text";
//Set the content type of the mail as TEXT_PLAIN or TEXT_HTML.
messageRequest.contentType = gmail:TEXT_PLAIN;
//Send the message.
var sendMessageResponse = gmailEP->sendMessage(userId, messageRequest);

The response from sendMessage is either a string tuple with the message ID and thread ID (if the message was sent successfully) or a GmailError (if the message was unsuccessful). The match operation can be used to handle the response if an error occurs.

string messageId;
string threadId;
match sendMessageResponse {
    (string, string) sendStatus => {
        //If successful, returns the message ID and thread ID.
        (messageId, threadId) = sendStatus;
        io:println("Sent Message ID: " + messageId);
        io:println("Sent Thread ID: " + threadId);
    }
    
    //Unsuccessful attempts return a Gmail error.
    gmail:GmailError e => io:println(e); 
}

The readMessage function reads messages. It returns the Message struct when successful and GmailError when unsuccessful.

var response = gmailEP->readMessage(userId, untaint messageId);
match response {
    gmail:Message m => io:println("Sent Message: " + m);
    gmail:GmailError e => io:println(e);
} 

The deleteMessage function deletes messages. It returns a GmailError when unsuccessful.

var delete = gmailEP->deleteMessage(userId, untaint messageId);
match delete {
    boolean success => io:println("Message deletion success!");
    gmail:GmailError e => io:println(e);
}
