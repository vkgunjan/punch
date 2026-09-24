# Office Punch Register

A mobile web app the security guard uses in place of the paper register to record punch in and punch out.

- **Punch**: search for an employee, then tap **Punch IN** or **Punch OUT**. The date and time are saved automatically. "Inside now" lists everyone who hasn't left yet.
- **Register**: shows the punches for any day, with Present, Inside and Left counts. You can delete a wrong entry, or enter a missed punch-out for an earlier day. A manually entered time is marked "edited".
- **Reports**: pick a date range and an employee, or all employees. Then:
  - **Download Excel** gives a `.xlsx` file with two sheets: *Punches* and a *Summary* of days present and hours per employee.
  - **Print** prints a register-style page with signature lines.
  - **Email / Share** opens the phone's share sheet with the Excel file attached, so you can send it with Gmail, WhatsApp and so on. On a desktop it downloads the file and opens an email draft.
- **Users**: add, edit or remove employees (name, code, department, mobile), or add many at once with **Import CSV** (columns: Name, Emp Code, Department, Mobile; tap **Download sample** for a template). Removing someone keeps their old punches in the reports.
- **Settings**: set the guard name, the report email and the GitHub cloud backup, make or restore a backup, and set the admin PIN.
- **Admin PIN** (optional): once set, the PIN is needed to open Settings, delete or correct punches, and edit or remove employees. The guard can still punch in and out and add new employees without it. After the PIN is entered, it isn't asked again for 5 minutes. The PIN is saved only on that phone, as a hash. If it's forgotten, clear the site data in Chrome to reset the app, then sync to get the data back from GitHub.

It also works offline. Punches are saved on the phone right away and upload once the internet is back.

## Where the data is saved

1. **On the guard's phone** (browser storage), as soon as each punch is made.
2. **In GitHub as JSON** (optional, but recommended) after a GitHub token is added in Settings:

```
data/users.json                  employee list
data/punches/2026-09.json        one file per month, grouped by day
```

Example of `data/punches/2026-09.json`:

```json
{
 "month": "2026-09",
 "days": {
  "2026-09-24": [
   { "id": "...", "userId": "...", "name": "Ramesh Kumar", "code": "E101", "dept": "Accounts",
     "date": "2026-09-24", "in": "2026-09-24T03:35:12.000Z", "out": "2026-09-24T12:31:05.000Z", "by": "Main Gate" }
  ]
 }
}
```

Times are stored in UTC (ISO format) and shown in local time in the app and in reports. Every change is a git commit, so there is a full audit history of who changed what and when.

If more than one phone uses the app with the same token, their data is merged automatically. The app syncs every minute.

## Setup

### 1. Publish the app (GitHub Pages)

```bash
git init
git add .
git commit -m "Office punch register app"
git branch -M main
git remote add origin https://github.com/vkgunjan/punch.git
git push -u origin main
```

Next, on GitHub go to the repo → **Settings → Pages** → Source: *Deploy from a branch* → Branch: `main`, folder `/ (root)` → Save.
> **The `punch` repo must be Public** for GitHub Pages to work on a free account (Settings → General → Danger Zone → Change visibility). The app code has no secrets in it. Keep the attendance data in a separate private repo, as described below.

After a minute the app is live at **https://vkgunjan.github.io/punch/**.

On the guard's phone, open that link in Chrome, then tap the ⋮ menu → **Add to Home screen**. It then opens like a normal app.

### 2. Keep the attendance data private (recommended)

On a free GitHub account, a Pages site has to be in a **public** repo, so anything saved in `vkgunjan/punch` can be seen by anyone.
Employee names and mobile numbers shouldn't be public, so:

1. Create a second, **private** repository, for example `vkgunjan/punch-data`, and tick "Add a README" so it isn't empty.
2. In the app's Settings, set **Data repository** to `punch-data`.

### 3. Create the GitHub token

1. On GitHub, open **Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token**.
2. Name it `punch-guard-phone` and set an expiry (for example 1 year).
3. **Repository access**: *Only select repositories* → choose the data repo (`punch-data`).
4. **Permissions → Repository permissions → Contents: Read and write**.
5. Generate it, copy the token, and paste it into the app under **Settings → Access token** → **Save settings**.

The badge at the top right should turn green and show **Synced**. The token is saved only on that phone. Use a token that can only reach the data repo, and create a new one if the phone is lost.

## Daily use for the guard

1. When an employee arrives: **Punch** tab → type part of the name → tap the name → **Punch IN**.
2. When they leave: tap their name in the search list or under "Inside now" → **Punch OUT**.
3. For a new employee: **Users** tab → fill the form → **Save employee**.
4. At the end of the day: **Reports** → *Today* → **Email / Share** or **Print**.

## Files

| File | Purpose |
|------|---------|
| `index.html` | The whole app (HTML, CSS and JavaScript, no build step) |
| `sw.js` | Offline support |
| `manifest.webmanifest`, `icon.svg` | "Add to Home screen" |
