# jobdog
Job application alert app to detect and determine whether an email is a rejection or successful application

**Initial Idea - brain dump**

Ok I want to create an application that opens on startup of windows. It checks for 60 seconds if outlook is open, if after 60 second outlook is still not open the application (let's call it JobDog for now) will open outlook so that it can gain access to the emails. 

JobDog has now opened outlook and loads all emails since the last time it performed a check (it will need a way to store the last time and date it performed a check). Once it has loaded the new emails it checks the subject and body of the email one by one and determines if it is a positive or negative response (rejection or proceeding to next round). 

JobDog then performs a calculation to update a visual dashboard to show the metrics. JobDog should perform the checks every minute while it's open so that it catches any new emails to update the dashboard. It also creates a good sound for positive responses or bad sound for negative responses. I also want the app to record how many applications have been made (by doing similar checks of emails searching for key strings such as 'thanks for your application", "we will get back to you", "if your profile fits" etc).



**ChatGpt - Initial Response From Brain Dump**
Response 1
Your idea for JobDog is well-formed — and I’ll break it down into its components, explain what’s possible, and how you can build it effectively. This will give you a clear path forward.

✅ Summary of what JobDog should do
Let’s structure your requirements:

1️⃣ Startup behavior
Runs automatically on Windows startup.

Waits up to 60 seconds checking if Outlook is open.

If Outlook isn’t open → launch Outlook via the app.

2️⃣ Email processing
Once Outlook is open:

Load all emails since the last check.

Store the last check timestamp somewhere (file, DB, registry).

Parse each new email’s subject + body:

Classify as: positive / negative / application sent.

Update the dashboard with counts & metrics.

Play a sound depending on the classification.

Continues to check for new emails every 60 seconds while running.

✅ Core components you'll need
Here’s the technical structure:

🚀 Windows startup
Make JobDog a desktop app (e.g., C#, Python, C++).

Add a shortcut to C:\Users\<you>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup to launch on boot.

📧 Detect Outlook running + open Outlook
C#:

csharp
Copy
Edit
using System.Diagnostics;
var outlookProcesses = Process.GetProcessesByName("OUTLOOK");
if (outlookProcesses.Length == 0)
{
    Process.Start("outlook.exe");
}
Wait up to 60 seconds, checking every few seconds if Outlook started.

📥 Access emails
✅ Since you want to interact with Outlook itself (not server API), you can:

Use Outlook Interop (COM) in C# or VB.NET:

Interact with the running Outlook instance.

Access Inbox, Other, Junk folders.

Read mail items (subject, body).

⚠ Outlook must be open for Interop to work (which is why JobDog opens it).

⏲ Track last check
Store timestamp of last check in:

A small local file (e.g., last_check.txt)

Or Windows Registry

Or a tiny SQLite database

On each run:

Load this timestamp.

Query Outlook folders for MailItem.ReceivedTime > lastCheckTime.

🧠 Classify emails
Build keyword rules:

csharp
Copy
Edit
string[] positiveKeywords = { "interview", "congratulations", "next step", "proceed" };
string[] negativeKeywords = { "regret", "unfortunately", "position filled", "not selected" };
string[] appliedKeywords = { "thank you for your application", "we will get back", "if your profile fits" };
For each email: lower-case subject + body, match keywords.

📊 Dashboard
Build a simple Windows Forms or WPF app:

Show counts: Applications Sent, Positive Responses, Rejections

Update metrics live.

You could graph these over time using charts.

🔉 Play sounds
Use System.Media.SoundPlayer in C# or playsound in Python.

Link positive/negative classification to different .wav files.

✅ Tech stack suggestion
🎯 Best fit for this project:
C# + .NET (Windows desktop app)
➡ Natural integration with Outlook Interop.
➡ Easy to manage startup tasks.
➡ Native UI (WinForms/WPF) + sound + charts.

✅ Challenges you should plan for
⚠ Outlook Interop depends on Outlook staying open — if the user closes it manually, JobDog will lose access.
⚠ If Outlook is slow to start, you’ll need to handle that gracefully.
⚠ Handling large volumes of mail — consider limiting to the last N days in searches.

📝 Development steps
1️⃣ Build a prototype that:

Detects Outlook

Opens it if needed

Lists new emails since last run

2️⃣ Add classification + sound + dashboard.

3️⃣ Package it to run on startup.

🚀 Alternative
If you want Outlook-independent access (no need to launch Outlook):
➡ Use Microsoft Graph API
➡ Build JobDog as a background app or service
➡ Requires registering your app in Azure AD.

If you want, I can:
✅ Provide example C# code for checking Outlook + reading new emails.
✅ Help you design the file format or DB for storing last check info.
✅ Help you mock up the dashboard UI.

Would you like to start with the C# skeleton code?





Response 2
That’s a solid, detailed plan — you’ve defined a complete workflow from launch to dashboard updates. Here’s a practical breakdown of what you’re proposing, what’s technically feasible, and how you could implement it step by step:

✅ 1) Launch on Windows startup
JobDog can register itself to run at Windows startup by adding a shortcut to the Windows Startup folder or setting a registry key (common in apps built with C#, Python, etc.).

On launch, JobDog runs in the background.

✅ 2) Check if Outlook is open
Every second for up to 60 seconds, JobDog checks running processes for OUTLOOK.EXE.

In C#, you could do:

csharp
Copy
Edit
var outlookRunning = Process.GetProcessesByName("OUTLOOK").Any();
In Python (using psutil):

python
Copy
Edit
import psutil
outlook_running = any("OUTLOOK.EXE" in p.name() for p in psutil.process_iter())
After 60 seconds:

If Outlook is not open, launch it with Process.Start("outlook.exe").

✅ 3) Connect to Outlook data
Since you plan to rely on Outlook being open, you can use the Outlook COM API via a library like:

C#: Microsoft.Office.Interop.Outlook

Python: pywin32 (win32com.client)

✅ This approach uses the local Outlook client to access cached emails (OST/PST), and it does not need server API calls or internet beyond Outlook itself.

✅ 4) Store the last check time
Save a timestamp locally (e.g., JSON, SQLite, registry) whenever JobDog completes a check:

json
Copy
Edit
{"last_check": "2025-06-23T14:15:00Z"}
On the next run, load this timestamp to fetch only emails received since then.

✅ 5) Load new emails since last check
Use the Outlook COM API to query your Inbox (and other folders like Junk, Other) with a filter on the ReceivedTime property greater than your last check.

Example in Python COM:

python
Copy
Edit
items = inbox.Items
items.Sort("[ReceivedTime]", True)
restriction = "[ReceivedTime] >= '" + last_check_time.strftime("%m/%d/%Y %H:%M:%S") + "'"
new_emails = items.Restrict(restriction)
✅ 6) Classify emails
For each new email:

Read subject and body.

Match against your keyword lists:

Negative: "regret", "unfortunately", "position filled", etc.

Positive: "interview", "congratulations", "next step", etc.

Application sent: "thank you for applying", "application received", etc.

Update metrics counters accordingly.

✅ 7) Update the visual dashboard
Build a simple GUI with:

A bar chart or counters for:

Applications sent.

Rejections.

Positive responses.

Libraries you could use:

Python: Tkinter, PyQt, or Kivy for GUI; matplotlib or plotly for charts.

C#: Windows Forms or WPF.

Refresh the dashboard every minute after new checks.

✅ 8) Play sounds on result
After each email check:

Play a positive sound for good news.

Play a negative sound for rejections.

Use system audio libraries:

Python: winsound or pygame.

C#: System.Media.SoundPlayer.

✅ 9) Keep JobDog running
Use a background loop with time.sleep(60) or a scheduled timer so JobDog performs checks every minute.

🔎 Key considerations
✔️ Why rely on Outlook being open?

The COM approach depends on the local Outlook instance.

If you instead used Microsoft Graph API, you wouldn’t need Outlook to be open — but you’d need to handle authentication separately (OAuth), and possibly refresh tokens.

✔️ Data storage

SQLite or a JSON file is sufficient for local persistence of last check timestamps and counters.

✔️ Performance

Email scanning should process only recent emails, not the entire mailbox each time, to avoid unnecessary CPU load.

✔️ Folders

If you want to include Other and Junk, you must specifically navigate to those folders in the Outlook object model.

✅ Summary: How JobDog will work
1️⃣ Launch on startup → check if Outlook is running → open Outlook if needed.
2️⃣ Once Outlook is open, connect via COM API.
3️⃣ Load emails received since the last check.
4️⃣ Classify emails → update counters → refresh dashboard → play sounds.
5️⃣ Repeat check every 60 seconds.
