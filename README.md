<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Joud Core - Blue Lock System</title>
    <style>
        :root { --blue: #00aaff; --red: #ff4444; --bg: #050505; --card: #0f0f0f; --text: #e0e0e0; }
        * { box-sizing: border-box; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
        body { background: var(--bg); color: var(--text); margin: 0; overflow: hidden; height: 100vh; }
        
        /* UI Elements */
        #main-ui { height: 100%; display: flex; flex-direction: column; }
        .top-nav { background: #000; padding: 15px; display: flex; justify-content: space-between; align-items: center; border-bottom: 2px solid var(--blue); }
        .logo { font-weight: bold; color: var(--blue); letter-spacing: 2px; font-size: 20px; }
        
        #main-menu { display: flex; flex-wrap: wrap; gap: 15px; padding: 20px; justify-content: center; overflow-y: auto; }
        .menu-card { background: var(--card); border: 1px solid #222; width: 150px; height: 150px; display: flex; flex-direction: column; align-items: center; justify-content: center; border-radius: 12px; cursor: pointer; transition: 0.3s; }
        .menu-card:hover { border-color: var(--blue); transform: translateY(-5px); box-shadow: 0 5px 15px rgba(0,170,255,0.2); }
        .menu-card i { font-size: 30px; margin-bottom: 10px; color: var(--blue); }

        #page-view { display: none; padding: 20px; height: calc(100% - 70px); overflow-y: auto; background: #080808; }
        .entry-item { background: #111; margin-bottom: 10px; border-radius: 8px; border-left: 4px solid var(--blue); overflow: hidden; }
        .entry-header { padding: 15px; display: flex; justify-content: space-between; cursor: pointer; background: #161616; font-weight: bold; }
        .entry-content { padding: 15px; display: none; line-height: 1.6; border-top: 1px solid #222; }
        
        /* Admin & Custom Tags */
        .bracket-text { color: #888; font-size: 0.9em; }
        .word-banned { color: var(--red); font-weight: bold; }
        .word-allowed { color: #44ff44; font-weight: bold; }
        .word-fg { color: #ffaa00; font-weight: bold; }
        .header-type { color: var(--blue); display: block; margin-top: 10px; font-weight: bold; border-bottom: 1px solid #333; }
        .dash-type { color: #bbb; display: block; padding-left: 10px; }
        .rule-img { width: 100%; border-radius: 8px; margin: 10px 0; border: 1px solid #333; }

        /* Modals */
        .modal { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.9); z-index: 1000; justify-content: center; align-items: center; }
        .modal-content { background: #111; padding: 25px; border-radius: 15px; width: 90%; max-width: 400px; border: 1px solid var(--blue); }
        input, textarea, select { width: 100%; padding: 12px; margin: 10px 0; background: #1a1a1a; border: 1px solid #333; color: #fff; border-radius: 5px; }
        .btn { background: var(--blue); color: #000; border: none; padding: 12px; width: 100%; border-radius: 5px; font-weight: bold; cursor: pointer; }
        
        /* Profile Dropdown */
        .profile-area { position: relative; display: flex; align-items: center; gap: 10px; cursor: pointer; }
        .avatar-img { width: 35px; height: 35px; border-radius: 50%; border: 2px solid var(--blue); object-fit: cover; }
        .dropdown-menu { display: none; position: absolute; top: 110%; right: 0; background: #111; border: 1px solid #333; border-radius: 8px; width: 150px; z-index: 100; }
        .dropdown-item { padding: 12px; border-bottom: 1px solid #222; font-size: 14px; }
        .dropdown-item:hover { background: #1a1a1a; color: var(--blue); }

        .dragging { opacity: 0.5; background: #222 !important; }
        .bl-footer { text-align: center; padding: 40px; color: #222; font-size: 40px; font-weight: 900; text-transform: uppercase; user-select: none; }
        .notify { position: fixed; bottom: 20px; left: 50%; transform: translateX(-50%); padding: 10px 20px; border-radius: 20px; font-size: 13px; z-index: 2000; display: none; }
    </style>
</head>
<body>

    <div id="main-ui">
        <div class="top-nav">
            <div style="display: flex; align-items: center; gap: 10px;">
                <span id="back-nav-btn" style="visibility: hidden; cursor: pointer; font-size: 20px;" onclick="closePage()">←</span>
                <div class="logo">JOUD CORE</div>
            </div>
            <div id="auth-section">
                <button class="btn" style="width: auto; padding: 5px 15px;" onclick="openAuthModal('login')">Login</button>
            </div>
            <div id="user-profile" class="profile-area" style="display: none;" onclick="toggleProfileDropdown(event)">
                <span id="user-nick" style="font-weight: bold;"></span>
                <img id="user-avatar" class="avatar-img" src="">
                <div id="profile-dropdown-menu" class="dropdown-menu">
                    <div class="dropdown-item" onclick="toggleProfileEdit()">Edit Profile</div>
                    <div class="dropdown-item" onclick="toggleStaffStats(true)">Staff Stats</div>
                    <div class="dropdown-item" style="color:var(--red)" onclick="logout()">Logout</div>
                </div>
            </div>
        </div>

        <div id="main-menu">
            <div class="menu-card" onclick="openPage('Style Rules')"><i>⚽</i><span>Style Rules</span></div>
            <div class="menu-card" onclick="openPage('Flow Rules')"><i>🌊</i><span>Flow Rules</span></div>
            <div class="menu-card" onclick="openPage('Global News & Archive')"><i>📰</i><span>Archive</span></div>
        </div>

        <div id="page-view">
            <h2 id="page-title" style="color:var(--blue)">Category</h2>
            <input type="text" id="rule-search" placeholder="Search rules..." oninput="renderEntries()">
            
            <div id="admin-panel" style="display: none; background: #111; padding: 15px; border-radius: 10px; margin-bottom: 20px;">
                <textarea id="text-input" placeholder="Type rule here... (Use # for title)"></textarea>
                <button class="btn" onclick="saveData()">Save Entry</button>
            </div>

            <div id="data-list"></div>
        </div>
    </div>

    <div id="auth-modal" class="modal">
        <div class="modal-content">
            <h3 id="auth-title">System Access</h3>
            <div id="auth-main-fields">
                <input type="text" id="auth-user" placeholder="Username">
                <input type="password" id="auth-pass" placeholder="Password">
                <button class="btn" onclick="processAuth()">Verify</button>
                <p onclick="switchAuthMode()" style="text-align:center; font-size:12px; margin-top:15px; cursor:pointer; color:var(--blue)">Switch Login/Create</p>
            </div>
            <div id="auth-2fa-fields" style="display:none">
                <input type="text" id="auth-2fa" placeholder="Enter 2FA Key">
                <button class="btn" onclick="verify2FA()">Confirm</button>
            </div>
            <button class="btn" style="background:none; color:#555; margin-top:5px;" onclick="closeAuthModal()">Cancel</button>
        </div>
    </div>

    <div id="backup-modal" class="modal">
        <div class="modal-content">
            <h3>System Backup</h3>
            <textarea id="backup-key-box" style="height: 100px; font-size: 10px;"></textarea>
            <button class="btn" onclick="restoreBackup()">Restore Data</button>
            <button class="btn" style="background:#333; margin-top:5px;" onclick="document.getElementById('backup-modal').style.display='none'">Close</button>
        </div>
    </div>

    <div id="reorder-overlay" class="modal" style="background:rgba(0,0,0,0.95)">
        <div class="modal-content" style="max-width:500px">
            <h3>Reorder Mode (F4)</h3>
            <div id="reorder-list" style="max-height:400px; overflow-y:auto"></div>
            <button class="btn" style="margin-top:20px" onclick="dragMode=false; document.getElementById('reorder-overlay').style.display='none'; renderEntries();">Finish & Save</button>
        </div>
    </div>

    <div id="notification" class="notify"></div>

    <script>
        // --- DATA CORE ---
        let joudDB = JSON.parse(localStorage.getItem('joud_core')) || { "Style Rules": [], "Flow Rules": [], "Global News & Archive": [] };
        let accounts = JSON.parse(localStorage.getItem('sys_accounts')) || [];
        let customRanks = JSON.parse(localStorage.getItem('sys_ranks')) || [{name: "OWNER", perm: "all"}, {name: "ADMIN", perm: "all"}, {name: "MEMBER", perm: "none"}];
        
        let currentUser = null;
        let activeCategory = "";
        let editIndex = -1;
        let authMode = "login";
        let dragMode = false;
        let spyTab = "users";

        // --- THE INJECTION KEY (Your F7 Code) ---
        const INJECTION_KEY = "eyJkYiI6eyJTdHlsZSBSdWxlcyI6WyIjIExpbWl0ZWQgU3R5bGVzXG5cbktyYW1wb3MgQmFyb3UgQWxsb3dlZDpcbi1DcmFzaCBTdGVhbDsgQWxsb3dlZCBjb21wbGV0ZWx5LCAgaWxsZWdhbCB0byB1c2UgaW50byBvciBpbnNpZGUgeW91ciBvd24gYm94ICggY291bnQgYXMgYSBkZWZlbnNpdmUgbW92ZSksIGlmIHVzZWQgaW50byBvciBpbnNpZGUgdXIgb3duIGJveCwgcmVzdWx0cyBpbnRvIGEgRkcgaWYgaXQgc3RvcHBlZCBhIGdvYWwsIGJhbm5lZCB0byBzdGVhbCAtbW92ZSBmcm9tIG9wcG9uZW50IGJveFxuLUJsaXp6YXJkIEZhbmcgU2hvdDsgQWxsb3dlZCBhbnl3aGVyZSBleGNlcHQgaW5ib3guICBBbGxvd2VkIHdpdGggbWV0YXZpc2lvbiAvIHByZWRhdG9yIGthcmFtcHVzIGZyb20gc2Vjb25kIGxpbmUgLiAob2cga2Fpc2VyIHJhbmdlKVxuLUtyYW1wdXMgQ2hvcCBEcmliYmxlOyBiYW5uZWQgaW50by9pbiBib3ggY2Fubm90IHVzZSBzaG90IGFmdGVyIHVudGlsIHlvdSBsb3NlIHRoZSBiYWxsXG4tUHJlZGF0b3IgS3JhbXB1cy9NZXRhdmlzaW9uO0Nhbm5vdCB1c2Ugc3BlZWQgaW5jcmVhc2VkIGZsb3dzIHdpdGggaXQuIENhbm5vdCB1c2UgYmxpenphcmQgZmFuZyBzaG90IHdpdGggaXRcbi1pZiB1IGFpa3UgZm9yY2UgZGVmZW5zZSBrcmFtcHVzIGJhcm91IGFuZCBpdCBidWdzIGluIGdvYWwgYW55d2F5cyB0aGVuIGl0cyBhIGZyZWVnb2FsXG5cbkVsZiBFbXBlcm9yIEFsbG93ZWQ6XG4tRWxmIEltcGFjdDsgYWxsb3dlZCBhbnl3aGVyZSBleGNlcHQgaW5zaWRlIG9mIHRoZSBvcHBvbmVudOKAmXMgYm94XG4tSGVscGVyIERyaWJibGU7IEJBTk5FRCBpbnRvIGJveCwgYWxsb3dlZCB0byBiZSB1c2UsedXNlZCB0d2ljZSAtRkxGIEVNUEVST1IgVk9MTEVZOyBpcyBiYW5uZWRcbi1FbGYgSW1wYWN0IHdoaWxlIHByb2RpZ3kgaXMgb24gaXMgYmFubmVkXG5cblNvYnplcm8gTG9raSBBbGxvd2VkOlxuLXxBUyBDTXxcbi1BaXIgdHJhcCBpbmYgb3QgKHJlbyBnZXRzIDQrMSBjYW1vcykgYW5kIGR0cmFwIG5hZ2kgYmFubmVkIHdpdGggaXRcbi1BaXIgdHJhcCBiYW5uZWQgdG8gZ2xvYmFsIHRyYXBcbi1BaXIgdHJhcCBub3QgYWxsb3dlZCBoZSBtYWtlIE0xcyBnaG9zdCBvbiBjaGlnaXJpIHNwZWVkc3RlciAod2lsbCBjb3VudCBhcyByYilcbi1BaXIgdHJhcCBhbGxvd2VkIHRvIG1ha2UgTTFzIGdob3N0IGluIG90aGVyIGNpcmN1bXN0YW5jZXMgKGNvdW50cyBhcyBhIHVzYWdlKVxuLU5lbCBSZW8gYWxsb3dlZCB3aXRoIGl0IGJ1dCBpZiBib3RoIG9uIHRoZSBwaXRjaCB3aXRoIExva2kgY20gdGhlbiBubyBvdGhlciBtYWpvclxuXG5TYW50YSBMYXZpbmhvdSBBbGxvd2VkOlxuLVdSQUQUEVEIERSSUJCTEU7IGJhbm5lZCBpbiBhbGxvd2VkIGludG8gYm94LSBcbi0gUFJFU0VOVCBTVFJJS0U7IGFsbG93ZWQgYW55d2hlcmUgYnV0IGJveCByZWFzb247IG5vdCBhcyBmYXN0IGFzIGtyYW1wdXMgYW5kIEthaSArIGNhbid0IGdyYXNzY3V0ICsgY2FuJ3QgYm91bmNlICsgbm8gaWZyYW1lcyBhZnRlciBzbyBVIGNhbiBnb2FsdGVuZCBpdC1cbi1TTEVJR0ggUklERTsgbm8gcmVzdHJpY3Rpb25zIChidW5zKVxuLUNIUklTVE1BUyBTVEFNUEVERTsgQWxsb3dlZCBhcyBkcmliYmxlIGJhbm5lZCBpbi9pbnRvIGJveFxuXG5EZW1vbiBTaGlkb3UgQWxsb3dlZDpcbi1EZW1vbmljIGRyaWJibGUgYWxsb3dlZCB0byBnbyBpbnRvIGJveCBhbmQgYWxsb3dlZCB0byB1c2UgaW5ib3hcbi1EZW1vbmljIHNob3QgYWxsb3dlZCBldmVyeXdoZXJlIGV4Y2VwdCBib3hcbi1EZW1vbmljIG92ZXJoZWFkIEtpY2sgYmFubmVkIGluYm94IGFsbG93ZWQgZXZlcnl3aGVyZSBlbHNlXG4tRGVtb25pYyBIZWFkZXIgYWxsb3dlZCBldmVyeXdoZXJlIGV4Y2VwdCBib3hcbi1CbG9vZGx1c3QgQWxsb3dlZCBvbmx5IGluIHlvdXIgaGFsZiB3aXRoIGluZmluaXRlIHVzZXNcbi1DYW4gdXNlIGl0IHRvIGdldCBvcGVuIGluIG9mZmVuc2l2ZSBzaWRlXG4tSW5mZXJubyBEcmFnb24gRHJpdmUgb25seSBhbGxvd2VkIHRvIGJlIHVzZWQgYXMgYSBzdGVhbCwgdXNpbmcgaXQgYXMgYSBzaG90IGlzIElMTEVHQUxcblxuUmVhcGVyIFNhZSBBbGxvd2VkOlxuLXNjeXRoZSBzbGFtIG9ubHkgYWxsb3dlZCAzIHRpbWVzIHBlciBsZWcsIG1pc3NlZCBzY3l0aGUgc2xhbXMgd2lsbCBiZSBjb3VudCBzZWQgKzEgb3Rcbi1wZXJmZWN0IHBhc3MgdXNhZ2UgSU5GSU5JVEUgLCBvbmx5IGFsbG93ZWQgdG8gcGxheWVyIG9uIHRoZSBzYW1lIGhhbGYgb2YgdGhlIGZpZWxkXG5cblNrZWxldG9uIE5hZ2kgQWxsb3dlZDpcbi1Ta2VsZXRvbiB0cmFwOyBub3QgYWxsb3dlZCB3aXRoIG5hZ2lcbi1SZW8gY2FudCBjb3B5IHVubGVzcyBubyBuYWdpIGluIHRlYW1cbi1LVUxMIFJPVUxFVFRFOyBiZWhpbmQgc2VtaSBjaXJjbGUgc2FtZSBhcyBkZW1vbmljIGhlYWRlclxuXG5CbG9vZG1vbiBSaW4gQWxsb3dlZDpcbi1DcmVzY2VudCBTaG90IGlzIEJBTk5FRCBpbmJveFxuLUx1bmFyIEF4aXMgYWxsb3dlZCAzeCBwZXIgbGVnXG4tQmxvb2QgZHJpYmJsZSBBTkQgQ3JvdyBpbGx1c2lvbiBhcmUgYm90aCBBTExPV0VEIGluIGFuZCBpbnRvIGJveFxuXG5QaGFudG9tIElzYWdpIEFsbG93ZWQ6XG4tTm8gcnVsZXMgZXhjZXB0IHNob3Qgc3RhY2sgaXNhZ2kgc2hvdCBpcyBiYW5uZWRcblxuRmlyZXdvcmsgQmFjaGlyYSBBbGxvd2VkOlxuLUV4cGxvc2l2ZSBkcmliYmxpbmcgKEMpIGJhbm5lZCBpbiBib3ogYWxsb3dlZCBpbnRvIGJveFxuLUZyZWFreSBzaG90IChmbGFzaHkgc2hvdCk6IEFsbG93ZWQgYW55d2hlcmUgYnV0IGluYm94IFxuLVJvY2tldCBwYXNzIChWKSBiYW5uZWQgaW4vaW50byBib3ggaXQncyBraW5kYSBidW5zXG4tSWxsdXNpdmUgRmlyZXdvcmtzIChCKSB1bnJlc3RyaWN0ZWRcblxuR2luZXJicmVhZCBDaGFybGVzcyBBbGxvd2VkOiBcbi1HaW5nZXJicmVhZCBkcmliYmxlOyBCQU5ORUQgaW4vaW50byBib3hcbi1Db29raWUgQ3Jvc3MgYW5kIEdpbmdlcmJyZWFkIENhbm5vbiA7IEJBTk5FRCBpbmJveCIsIiMgTWFzdGVyIFN0eWxlc1xuXG5Mb2tpIEJhbm5lZDpcblxuTGF2aW5ob3UgQWxsb3dlZDpcbi0gKG9ubHkgNCBkYXNoZXMgYWxsb3dlZCBwZXIgYXR0YWNrKVxuLSB0ZWNocyB0aGF0IGFsbG93IHlvdSB0byBkbyAyKyBnaW5nYSBzdHlsZSBkYXNoZXMgYXQgb25jZSBhcmUgYmFubmVkXG4tIFJpc2luZyBkYW5jZSBiYW5uZWQgaW4vaW50byBib3hcbi0gQnV0dGVyZmx5IGNvdW50ZXIgYmFubmVkIGluYm94L2ludG9cbi0gQXdha2VuaW5nO1xuLSBUaWtpIFRha2EgYmFubmVkIGluL2ludG8gYm94XG4tIEJ1dHRlcmZseSBkYW5jZSBpcyBiYW5uZWRcbi0gYmVlZnJlZXN0eWxlIC8gYnV0dGVyZmx5ZGFuY2VyIGJhbm5lZCB3aXRoIGxhdmluaG8gZmxvd1xuLSBCdXR0ZXJmbHkgaW50ZXJjZXB0aW9uIGFsbG93ZWQgb25seSBpbiB5b3VyIGhhbGZcbi0gR2luZ2Egc2hvdCBhbGxvd2VkIG9ubHkgb3V0IG9mIGJveFxuIiwiIyBHZW5lcmF0aW9uYWwgU3R5bGVzXG5cblNhZSBBbGxvd2VkOiAobWFqb3IgZGVmZW5zaXZlKVxuLSBPZmYtQmFsYW5jZSBpbmZpbml0ZSB1c2VzIChiYW5uZWQgdyBuZWwgcmVvICkgT2ZmLSBCYWxhbmNlIGFsbG93ZWQgb2ZmZW5zaXZlbHkuIDMgdGltZXMgb25seSBtaXNzZWQgZG9udCBjb3VudCAoIGJhbm5lZCB0byB1c2Ugb24gY20gKSBiYW5uZWQgaW4gb3Bwb3NpdGUgdGVhbSBib3ggb25seSAoYWxsb3dlZCBpbiBvd24pLlxuLSBQZXJmZWN0IFBhc3MgYmFubmVkIHRvd2FyZHMgYSBwbGF5ZXIgaW5zaWRlIG9wcG9uZW50IGJveCxcbiAtIFdoZW4gUGVyZmVjdCBQYXNzIGlzIHVzZWQgb24gYSBwbGF5ZXIgd2l0aCBhIHNob3QgdGhlIHBsYXllciBtdXN0IHdhaXQgMyBzZWNvbmRzIGJlZm9yZSBzaG9vdGluZyBlLmcgc2hpZG91XG4tIENyZWF0aXZlIGRyaWJibGUgYWxsb3dlZCBpbnRvIGJveCwgYmFubmVkIGluc2lkZSBib3guXG4tIFNsaWRlIHRlY2ggYmVhdXRpZnVsIHNob3QgaXMgYmFubmVkLlxuLSBEaXZpbmUgaW50ZXJ2ZW50aW9uIGlzIGJhbm5lZC5cbi0gYmVhdXRpZnVsIHNob3QgYWxsb3dlZCBpbmJveFxuLSB1c2luZyBtb3ZlcyB3aGlsZSBjb29sZG93biBmbG93IGlzIHBvcHBlZCBpcyBhbGxvd2VkXG5cbkthaXNlciBBbGxvd2VkOlxuIC0gS2Fpc2VyIEltcGFjdCBhbGxvd2VkIGFueXdoZXJlIGV4Y2VwdCBpbnNpZGUgb2YgdGhlIG9wcG9uZW504oCZcyBib3guICBcbi0gRW1wZXJvciByb3V0ZSBvZmYgYmFsbCBpcyBhbGxvd2VkLCBidXQgYmFubmVkIGluIGF3YWtlbmluZyBvbiBcbi0gU3ViamVnYXRpb24gaXMgYmFubmVkXG4tIFJveWFsIHZvbGxleSBpcyBiYW5uZWQgXG4tIEVhc3RlciBLYWlzZXIgc2tpbiBiYW5uZWQgdW5sZXNzIG9wcG9uZW50IHRlYW0gYWxsb3dzIHUuXG4tIEFueSB0ZWNoIHRvIHJlc2V0IEVtcGVyb3Igcm91dGUgY29vbGRvd24gaXMgYmFubmVkIFxuLSBLYWlzZXIgSW1wYWN0ICB3aGlsZSBwcm9kaWd5IGlzIG9uIGlzIGJhbm5lZFxuLSBLYWlzZXIgY2hlbXJlYWN0IGFsbG93ZWQgMyB0aW1lcyBwZXIgbGVnLiBDaGVtcmVhY3QgYWxsb3dlZCBvbmx5IG91dCB0aGUgYm94XG5cbkRvbiBMb3JlbiB6byBBbGxvd2VkIDpcbi0gQXMgQ007XG4tIEdvbGRlbiBkZWZlbnNlID4gb25seSBpbiB5b3VyIGhhbGYgb2YgdGhlIGZpZWxkXG4tIENhbiBiZSB1c2VkIGluIG9wcG9uZW50IGhhbGYgdG8gcGFzcyBvciBzaG9vdC5cbi0gWm9tYmllIHN0ZWFsIGJhbm5lZCBpbiB5b3VyIG93biBzbWFsbCBib3gsIGFsc28gYWxsb3dlZCBvbmx5IGluIHlvdXIgaGFsZi5cbi0gQXMgRFdJTkc7ICBiYW5uZWQgd2l0aCBjb29sZG93biByZWR1Y3Rpb24gZmxvd3Ncbi0gR29sZGVuIGRlZmVuc2UgMysxIG1pc3NlZCBkb250IGNvdW50IChiYW5uZWQgaW4gbWluaSBib3gpXG4tIFpvbWJpZSBzdZWFsIGJhbm5lZCBpbiB5b3VyIG93biBzbWFsbCBib3gsIGFsc28gYWxsb3dlZCBvbmx5IGluIHlvdXIgaGFsZlxuXG5CVU5OWTogXG4tIEJ1bm55IGhvcCBjYW5ub3QgYmUgdXNlZCBpbi9pbnRvIGJveC4gXG4tIEJ1bm55IHRyYXAgaXMgYmFubmVkLiBcbi0gRW50ZXIgc3RyaWtlciBpcyBiYW5uZWQuIFxuLSBKdW1wIGFuZCBmbHlpbmcgc2Npc3NvciB0ZWNoIGlzIGJhbm5lZC4gXG4tIFNjaXNzb3Iga2ljayBhbGxvd2VkIG9ubHkgaW4gdGhlIGdyZWVuIGFyZWEgYmVsb3c7XG5odHRwczovL2Nkbi5kaXNjb3JkYXBwLmNvbS9hdHRhY2htZW50cy8xNDM5MDA2MDAwNTIxNjc0ODg0LzE0NzQwNTI0MzA1MDQxMzY4ODkvOEM0MjdBQTktNDc2Ri00OTFGLThERTUtNjdGNEJDOTc4RDlBLnBuZz9leD02OTk4NzFjZSZpcz02OTk3MjA0ZSZobT00ODZiNTJiM2M1ZTdlNmRkZWFlN2JjYjY1MjM1ZmYxMGJjYzdidThhMjY5ZWI0MmYxYWY4NzYyMWI0Zjk3MTA5JiIsIiMgV29ybGRlIENsYXNzIFN0eWxlc1xuXG5OZWwgTmFnaSAvIE1jIE5hZ2kgRnJ5c2hpcm86XG4tIFRyYXBwaW5nIEdLIHBhc3NlcyBpcyBhbGxvd2VkXG4tIFplcm8gcmVzZXQgdHVybiBiYW5uZWQgaW5ib3hcbi0gbG9uZyBkYXNoIHRlY2ggaXMgYmFubmVkLlxuLSBjYW5ub3QgdXNlIHN0ZWFsIG1vdmVzIG9mIG5lbCBuYWdpIHdpdGggb3RoZXIgc3R5bGVzIHN0ZWFsIG1vdmVzICggdXNpbmcgb24gb3duIGJhbGwgdG8gYWN0aXZhdGUgc2hvdCBkb2VzbuKAmXQgY291bnQpXG4tIEp1Z2dsZSBmZWludCBhbGxvd2VkIHR3aWNlIHBlciBhdHRhY2tcbi0gTW92ZXN0YWNrIGlzIGJhbm5lZFxuLSBKdWdnbGUgZmVpbnQgYWxsb3dlZCBkYXJrIGxpbmUgYW5kIFRyYXAgc2hvdCBhbGxvd2VkIGZyb20gcmVkIGxpbmUgYXMgdGhlIHBpYyBiZWxvdztcbmh0dHBzOi8vY2RuLmRpc2NvcmRhcHAuY29tL2F0dGFjaG1lbnRzLzE0MzkwMDYwMDA1MjE2NzQ4ODQvMTQ3NDA1MzI4ODE4MDU4NDQ3MC83NDgyQTIwRi1DRkY4LTQ1NzMtOTJGMy1ENTM3MDYzRjFCMTgucG5nP2V4PTY5OTg3MjlhJmlzPTY5OTcyMTFhJmhtYzgxOWQwYzg5Y2Y4MzcxOTI1ZDQxNWUzMzI5ZmZjMzIxZTdhNjk2ZjIxNjc5NjA2NGYzNWQ4Njg3NzU5ODRkNiZcbkNoYXJsZXNzOlxuLSBvbmx5IG91dCB0aGUgYm94IEJhbm5lZCB3aXRoIGNkIHJlZHVjaW5nIEZsb3dzXG4tU2t5IGFyYyBwYXNzIGNhbiBiZSBvbmx5IHVzZWQgdG8gYSBwbGF5ZXIgaW4gdGhlIHNhbWUgaGFsZiBhcyB5b3UsIGlsbGVnYWwgdG8gcGFzcyB0byBhIHBsYXllciBpbi9nb2luZyBpbnRvIGJveC1cbi0gQ29udHJhcmlhbiBwYXNzIGlzIGFsbG93ZWRcbi0gQ29udHJhcmlhbiBkcmliYmxlIGJhbm5lZCBpbi9pbnRvIGJveFxuLSBD2250cmFyaWFuIGRlZmVuc2UgYWxsb3dlZCBvbmx5IGZvdXIgdGltZXMgcGVyIGxlZysxIG90KG1pc3NlZCBkb250IGNvdW50KSBiYW5uZWQgb24gaWZyYW1lIHNob3RzLCBhbmQgbWF5IGJlIHVzZWQgaW5maW5pdGVseSBhcyBhIGNtXG4tIHRyYWl0IHN0YWNraW5nIGlzIGJhbm5lZCBmb3IgZXhhbXBsZSB1c2luZyBnb2RzcGVlZCB3aGlsZSBtZXRhdmlzaW9uIGlzIG9uXG4tIEF3YWtlbmluZyBzaG90IGlzIGJhbm5lZCwgZGVmZW5zaXZlIGF3YWtlbmluZyBtb3ZlIGlzIGFsbG93ZWQgb25seSBpbiB5b3VyIGhhbGYoc2F2ZWRcbi0gQmFubmVkIHdpdGggTmVsIHJlb1xuXG5ORUwgSXNhZ2k6XG4tIE9mZmVuc2l2ZSBkYXNoIGlzIGFsbG93ZWQgb25seSAyIHRpbWVzIHBlciBhdHRhY2ssIGl0IGlzIGFsbG93ZWQgdG8gZ28gaW50byBib3gsIGJhbm5lZCB0byB1c2UgSU5TSURFIEJPWFxuLSBEZWZlbnNpdmUgZGFzaCBhbGxvd2VkIG9ubHkgaW4geW91ciBvd24gaGFsZlxuLSBORUwgaXNhZ2kgcGFzcyBpbnRvIG5hZ2kgdHJhcCAgaXMgbm90IGFsbG93ZWRcbi0gT21uaXZpc2lvbiBhbGxvd2VkIGFueXdoZXJlIGV4Y2VwdCBvcHBvbmVudCBib3gsIEJBTk5FRCBUTyBTQ09SRSBXSVRIXG4tIGJsaW5kIHNwb3QgY29tcGxldGVseSBiYW5uZWRcbi0gTkVMIGlzYWdpIGZha2UgcmFpbmJvdyBmbGljayBzaG90IGFsbG93ZWQgYnV0IGZha2Ugc2hvdCBpcyBub3QgYWxsb3dlZFxuLSBLYWlzZXIgY2hlbXJlYWN0IGFsbG93ZWQgMyB0aW1lcyBwZXIgbGVnLiBDaGVtcmVhY3QgYWxsb3dlZCBcbi0gT25seSBpbiB0aGUgZ3JlZW4gYXJlYSBiZWxvdCA7XG5odHRwczovL2Nkbi5kaXNjb3JkYXBwLmNvbS9hdHRhY2htZW50cy8xNDM5MDA2MDAwNTIxNjc0ODg0LzE0NzQwNTIxMDg4ODUxNjgzMTEvODU5NTc2QUUtN0IyOS00NjlGLUJDREMtREFGMEU2RjM2QURGLnBuZz9leD02OTk4NzE4MSZpcz02OTk3MjAwMSZobT0yZTFkNjAzNmE4Yjc1YTNkNmQyMzJjYjhmOTQzZjg4MWU5NTE5OTExZDM3NGFiZWMxMzU5Nzg4M2FjYWRjZDgxJiIsIiMgTXl0aGljIFN0eWxlc1xuXG5TaGlkb3UgQWxsb3dlZDpcbi1EZW1vbiBkcml2ZSBhbGxvd2VkIGludG8gYm94IGJ1dCBiYW5uZWQgaW5ib3ggKCBvcHBvbmVudCB2ZXJzaW9uKS1cbi1EZW1vbiBkcml2ZSBhbGxvd2VkIGludG8gYm94IGFuZCBpbiBib3ggKCBUZWFtIHBlbmFsdHkgYm94IHZlcnNpb24pIEhvbGRpbmcgbTEgLyB2b2xsZXkgYXQgdGhlIGVuZCBpcyBhbGxvd2VkIGJ1dCBub3QgYXQgdGhlIHN0YXItXG5cbk5lc3MgQWxsb3dlZDpcbi1NYWdpY2FsIFBhc3MgJiBpbGx1c2lvbiBDcm9zczsgYmFubmVkIGluL2ludG8gYm94LCBhbmQgYmFubmVkIGZyb20gaGFsZiB0byBoYWxmLlxuLSBXZWFwb24gRGlzYWJsZXI6IDQrMSBvdCBtaXNzZWQgZG9udCBjb3VudCwgYmFubmVkIGluIG9mZmVuc2l2ZSBoYWxmXG5cbkFpa3UgQWxsb3dlZDpcbi1EZWZlbnNpdmUgbW92ZXMgYWxsb3dlZCBvbmx5IGluIHlvdXIgaGFsZi4gKGlmcmFtaW5nIGd0IGZ1bGwgYmFuKVxuLSBUb3RhbCBjbGVhcmFuY2UgaXMgYmFubmVkIGludG8gYm94IGFuZCBpbnNpZGUgYm94LiBJdHMgIG9ubHkgYWxsb3dlZCA0IHRpbWVzIHBlciBsZWcsICsxIHVzZSBpbiBvdmVydGltZS4gTWlzc2VkIHVzZXMgY291bnQuIFVzaW5nIHRvdGFsIGNsZWFyYW5jZSBpbmJveCByZXN1bHRzIGluIHR1cm5vdmVyLCBidXQgaWYgdGhlIHBsYXllciBpdHMgYmVlbiB1c2VkIG9uIGhhZCBmbG93ICsgbW92ZXMgKCBvciB3YXMgY2xlYXJseSBpbmJveCBiZWZvcmUgdGhlIGNtIHVzZWQgbW92ZSkgaXQgcmVzdWx0cyBpbiBmcmVlIGdvYWwgYXMgaXQgc3RvcHBlZCBhIGdvYWwuIChpZiByZWZlcmVlIGRlY2lkZWQgaXQgaXMgd29ydGggKzEgaW5zdGVhZCBvZiB0dXJuIG92ZXIgdGhlbiBpdCBpcyArMSlcbi1TbmFrZSBpbnRlcmNlcHRpb24gYWxsb3dlZCAyIHRpbWVzIHBlciBsZWcgKzEgb3ZlcnRpbWVcbi1Vc2luZyBTbmFrZSBpbnRlcmNlcHRpb24gdG8gc3RvcCBOYWdpIGxvYiBpcyBhbGxvd2VkIGlmIHlvdSBhcmUgbm90IHVzaW5nIGRyYXAgb3IgYWlyIHJlbyBvbiBcbi1TbmFrZSBpbnRlcmNlcHRpb24gYWxsb3dlZCBvZmZlbnNpdmVseSB0byBzaG9vdCBidXQgY2Fubm90IHVzZSBpdCBwYXN0IHdoaXRlIGRvdCBpbmJveCwgc2xpZGUgdGVjaCBzaG90IGJhbm5lZC5cbi0gRm9yIGluYm94IHRvdGFsIGNsZWFyYW5jZSwgaWYgcGxheWVyIGlzIGluYm94IHdoaWxlIHlvdSBhcmUgb3V0c2lkZSBpdCBzdGlsbCBjb3VudHMgYXMgaW5ib3gsIGlmIGNtIGFpa3UgaXMgaW5ib3gvb24gbGluZSB3aGlsZSBwbGF5ZXIgb3V0c2lkZSBpdCBjb3VudHMgaW5ib3guXG4tIFVzaW5nIHRvdGFsICBjbGVhcmFuY2Ugb24gbW92ZW1lbnQgLyByb3RhdGlvbiBtb3ZlcyBpcyBhbGxvd2VkLCBmb3IgZXhhbXBsZSBkYXNoICsgc3BlZWRzdGVyIChhbmQgb24gc2hvdHMgYXN3ZWxsKVxuXG5ZdWtpbWl5YSBBbGxvd2VkOlxuLSBMYUJvYmEgJiBTdHJlZXQgc3dlZXA7IGNhbuKAmXQgdXNlIGFueSBidWdzIHRoYXQgY2hhbmdlcyBtb3ZlbWVudCwgY2Fu4oCZdCB1c2Ugc2xpZGUgdGVjaCBvbiB0aGVtXG4tIFN0cmVldCBzd2VlcCAtPiBhbGxvd2VkIGludG8gYW5kIGluc2lkZSB0aGUgYm94XG4tIExhQm9iYSAtPiBjYW4gYmUgdXNlZCBJTlRPICBBTkQgSU5CT1hcbi0gQXdha2VuaW5nIGRyaWJibGUgbW92ZSBhbmQgc2hvdCBtb3ZlOyBiYW5uZWQgdG8gYmUgdXNlZCBpbnNpZGUgdGhlIGJveCBvciBpbnRvIGl0XG4tIEd5cm8gc2hvdDsgQ2FuIGJlIHVzZWQgYW55d2hlcmVfbiBvbiBcblxuTmVsIFJpbiBBbGxvd2VkOlxuLSBOb3JtYWwgcmluOyBubyBcnVsZXMgc2hvdCBhbGxvd2VkIGluYm94XG4tIE5lbCByaW4gcnVsZXM6IHlldCB0byBiZSBkZWNpZGVkXG5cbk5lbCBSZW8gQWxsb3dlZCA6XG4tIFBvcHBpbmcgYW55IGNkIGZsb3cgZGlyZWN0bHkgYWZ0ZXIgY2Ftb3VmbGFnZSBpcyBiYW5uZWQgIChyZXN1bHQgaW4gaGFsZmluZyBjYW1vIGNkIGidWcpXG4tIE5vdCBhbGxvd2VkIHRvIGNvcHkgRG9uL0lnYWd1cmkvYWlrdS9TWmxva2kgYXdrIGlmIHlvdSBoYXZlIG9uZVxuLSBBbnkgZmxvdyB0aGF0IGluY3JlYXNlcyBzaG90IHBvd2VyIGlzIGJhbm5lZCB3aXRoIFN0cmlrZXIgTW9kZSBpbnNpZGUgYm94LCBNVVNUIENMSVAgRVZFUlkgR09BTCBTQ09SRUQgV0lUSCBORUwgUkVPIFdISUxFIEhBVklORyBGTE9XIFBPUFBFRC5cbi0gQ29weWluZyBEb24vSWdhZ3VyaS9haWt1L25hZ2kvU1psb2tpIHdoaWxlIGhhdmluZyBvbmUgb2YgdGhlc2Ugc3R5bGVzIG9uIHlvdXIgdGVhbSBpcyBiYW5uZWQuIChleGNlcHQgc25ha2UgaW50ZXJjZXB0aW9uIHRvIHVzZSBvZmZlbnNpdmVseSkgXG4tIElmIHVzZWQgYXMgY20geW91IGNhbiBlaXRoZXIgY2hvb3NlIGNvcHlpbmcgZmQvbG9raSBkdHJhcCBvciB1c2luZyBvZmYgYmFsbCB2IChTdGVhbHRoIFN0ZXBzL2NhbW91ZmxhZ2UvQ2hhbWVsZW9uIERlZmVuc2UpLCB1c2luZyBib3RoIGlzIGJhbm5lZC5cbi0gU3RlYWx0aCBTdGVwcy9jYW1vdWZsYWdlIGlzIGFsbG93ZWQgb25seSBpbiB5b3VyIGhhbGYoYmFubmVkIGluYm94LGFsbG93ZWQgaW50bykuIDQrMSBvdmVydGltZSBpbmZpbml0ZSBvbiBsb29zZSBiYWxsIGRvZXNu4oCZdCBjb3VudCBpZiBtaXNzZWQgYnV0IGNhbuKAmXQgdXNlIGl0IGludG8gdGhlIHNtYWxsIGJveCBjdXogaXQganVzdCBlYXRzIHRoZSBiYWxsIChldmVuIGlmIG5vMSBlbHNlIGlzIGdvYWwgdGVuZGluZylpZiBkb25lIHNvIGl04oCZcyBhIHR1cm5vdmVyICsgcmIvZmcgZGVwZW5kaW5nIG9uIHNpdHVhdGlvbiBpZiB1ciBjYW1vIHRvIHNtYWxsIG1pc3NlZCBhbmQgdXIgb25seSBndCB0aGVuIG5vIHJiIFxuLSBDYW50IHVzZSBvbiBpZnJhbWUgc2hvdHMgcnVsZWJyZWFrKyB0dXJuIGlmIGRvbmUsIGFuZCBmZyBzZWNvbmQgdGltZSwgd2hhdCB1IGNhbiBzbGlkZSBkb2VzbnQgY291bnQgYXMgaWZyYW1lIHNob3QgKGFsbG93ZWQgb24gcm91dGUsIGlmIHUgcm91dGUgYW5kIGluc3RhbnQgaW1wYWN0e2F2b2lkaW5nIHJiIGh1bnRpbmd9KVxuLSBTbGlkZSB0ZWNoIHRvIGdvIGZ1cnRoZXIgd2l0aCB2IG1vdmUgaXMgYmFubmVkIC4gIChhbnkgdGVjaCB0byByZXNldCBTdGVhbHRoIFN0ZXBzL2NhbW91ZmxhZ2UgY2QgaXMgIGJhbm5lZCApXG4tIFlvdSBjYW4gY29weSBBTExPV0VEIGRlZmVuc2l2ZXMgbW92ZXMgMyB0aW1lcyBwZXIgbGVnKGZvciBlYWNoIGRlZmVuc2l2ZSBtb3ZlcmFmYSBydWxlIGRvbnQganVtcCBtZSApLiBUaGUgZGVmZW5zaXZlIG1vdmVzIHlvdSBjYW4gY29weTsgKGFsbCBtaW5vciBkZWZlbnNpdmUgbW92ZXMpIChDb3B5aW5nIE1vdmVzIGZyb20gc3R5bGVzIGluIGxvYmJ5IG9yIGEgZGVmZW5zaXZlIG1vdmUgZnJvbSBhIHN0eWxlIGluIHlvdXIgdGVhbSAtIEFsbG93ZWQgaWYgaXRzIG5vdCB1c2luZyBtb3ZlIGRlZmVuc2l2ZWx5IChleGNlcHQgZ2FnYSkgaXMgaWxsZWdhbCkgXG4tIFJlbyBjb3B5aW5nIEthaXNlciBJbXBhY3QvRWxmIEltcGFjdCBpcyBpbGxlZ2FsLiBcbi0gQ2hhbWVsZW9uIERlZmVuc2UgKCBWIE9mZmJhbGwgQWlyIFZhcmlhbnQgKSBpcyBhbGxvd2VkIG9ubHkgMiB0aW1lcyBwZXIgbGVnIGFuZCArMSBpbiBvdmVydGltZSBcbi0gTmVsIFJlbyBjb3B5aW5nIGRyaWJibGUgbW92ZXMgdGhhdCBoYXMgc2hvdCBsaWtlIG5hZ2kgdHJhcCBvciBrYWlzZXIgZHJpYmJsZSBpcyBiYW5uZWQgaWYgdGhlIGNvcGllZCBzdHlsZSBpcyBpbiB0aGVpciB0ZWFtLiAodXNpbmcgdGhlIHRyYXAgY29weSBkZWZlbnNpdmVseSBpcyBiYW5uZWQgZXZlbiBpZiB5b3V donQgaHAVEIGEgbmFnaSBvbiB5b3VyIHRlYW0gYW5kIGRvbnQgdXNlIENoYW1lbGVvbiBEZWZlbnNlIClcbi0gVXNpbmcgbW92ZXMgd2hpbGUgYSBmbG93IHRoYXQgcmVkdWNlcyBjb29sZG93biBpcyBwb3BwZWQgb24gaXMgYmFubmVkLlxuLSBJZiB5b3XigJlyZSB1c2luZyB0aGUgZGVmZW5zaXZlIG1vdmVzIG9mIGEgc2Vjb25kIGRlZmVuc2l2ZSBzdHlsZSB5b3UgY2Fu4oCZdCBjb3B5IG9yIHVzZSBhbnkgZGVmZW5zaXZlIG1vdmVzIHdpdGggcmVvLlxuLSBJZiB5b3UgaGF2ZSBhIHdvcmxkIGNsYXNzIG9uIHlvdXIgdGVhbSB5b3UgY2Fu4oCZdCBjb3B5OyAodSBjYW4gY29weSBmcm9tIGFub3RoZXIgd2Mgd2hpbGUgaGF2aW5nIGEgd2MgdXJzZWxmICwgaWYgdGhlIHdjIHUgY29waWVkIGFuZCB3YyB5b3UgaGF2ZSBpcyBhbGxvd2VkIHRvIGJlIHBsYXllZCB0ZyAoY2FudCBjb3B5IHdvcmxkIGNsYXNzZXMgaW4geW91ciBvd24gdGVhbSB1bmxlc3MgaXQgd2FzIHN0YXRlZCB0byBiZSBhbGxvd2VkIGxpa2UgcmluIGRyaWJibGUpXG4tIGFubmloaWxhdGVcbi0ga2Fpc2VyIGRyaWJibGVyICYgaW1wYWN0XG4tIGRvbiBsb3JlbnpvIFxuLSBhbnkgb3RoZXIgYmFubmVkIG1vdmVzXG4tIGFueSB0eXBlIG9mIGJvZHlibG9jayAvIGFybXB1bGwvc2hhcmsgc3RlYWwgdG8gY29weSBpcyBiYW5uZWQiLCIjIExlZ2VuZGFyeSBTdHlsZXNcblxuTmFnaSBBbGxvd2VkOlxuLSBXYWxraW5nIHRyYXAsIGRhc2hpbmcgaW50byBzbWFsbCBnayBib3ggYXJlIGJhbm5lZFxuLSBNMSBkdXJpbmcgYW5pbWF0aW9uIG9mIHNsaWRlIHRlY2ggdHJhcCBpcyBiYW5uZWRcbi0gQ2FuIHVzZSBuYWdpIHZvbGxleSBpbiB5b3VyIGhhbGYgb25seVxuLSBEZWZlbnNpdmUgVHJhcCBiYW5uZWQgIGlmIE5lbCBSZW8gdGVhbW1hdGUgdXNlZCBBaXIgVmFyaWFudCBvZiBWIG1vdmVcbi0gR2xvYmFsIGNkICBjb3VudHMgYXMgYSBEZWZlbnNpdmUgVHJhcFxuLSBoaWRpbmcgVHJhcCBzaG90IGFuaW1hdGlvbiBpcyBiYW5uZWRcblxuTmVsIEJhY2hpcmEvQmFjaGlyYSBBbGxvd2VkOlxuLSBIZWVsIENlbnRlciBpbi9pbnRvIGJveCAtIEJhbm5lZFxuLSBJbnN0YSBzaG9vdGluZyBhZnRlciBIZWVsIENlbnRlciAtIElMTEVHQUwsIGlmIHNjb3JlZCwgb3Bwb3NpdGUgdGVhbSBpcyBhd2FyZGVkIGEgZnJlZSBnb2FsLiAoZXhhbXBsZTogS2Fpc2VyIEltcGFjdCwgTmFnaSBUcmFwIC0+IG1pZCBhaXIsIG5lbCByaW4gc2hvdClcbi0gUm91bGV0dGUgU3RlYWwgLSBhbGxvd2VkIHN0cmljdGx5IGRlZmVuc2l2ZSBzaWRlLCBpdCBjb3VudHMgYXMgZGVmZW5zaXZlIG1vdmVcbi0gQmFycmllciBzaG90cyBhcmUgYmFubmVkXG4tIHBhc3MgbXVzdCBiZSBvbmx5IHVzZWQgd2hlbiB5b3UgaGF2ZSB0aGUgYmFsbFxuXG5LYXJhc3UgQWxsb3dlZDpcbi0gQ3JvdyBzdGVhbCBiYW5uZWQgaW4gb3Bwb25lbnQgYm94XG5cbkt1bmlnYW1pIEFsbG93ZWQ6XG4tIEJvZHlibG9jayA+IGFsbG93ZWQgdG8gdXNlIGRlZmVuc2l2ZWx5IG9ubHkgaW4gb3duIHNpZGVcbi0gQm9keWJsb2NrIGNhbiBiZSB1c2VkIGFzIGEgZHJpYmJsZSBtb3ZlL29mZmVuc2l2ZWx5IGFueXdoZXJlIGV4Y2VwdCBvcHBvbmVudCBib3guLSBzaG90IHN0YWNrIGJvZHlibG9jayBhbmQgcG93ZXIgc2hvdCBub3QgYWxsb3dlZFxuLSBCb2R5IGJsb2NrIHRlY2ggd2hlcmUgdSBmbHkgaW50byB0aGUgYWlyIGFuZCBzaG9vdCBpcyBhbGxvd2VkIChleGNlcHQgaWYgdXNlZCBwb3dlciBzaG90KVxuLSBCb2R5YmxvY2sgYWxsb3dlZCBpbmJveCAoT1dOIEJPWCBOT1QgT1BQT05FTlQpXG4iLCIjIEVwaWMgU3R5bGVzXG5cbkt1cm9uYSBBbGxvd2VkOlxuLSBvbmUtdHdvIGluL2ludG8gYmlnIHBlbmFsdHkgYm94IGlzIGJhbm5lZFxuLSBTaGFya3N0ZWFsIDUrMSBpbiBvdCwgb25seSB1ciBzaWRlIHN0cmljdGx5LCBtaXNzZWQgZG9lc250IGNvdW50XG4tIG9uZS10d28gIHRvIGFueSBzdHlsZSBzaG90IGxpa2UgaW1wYWN0IGlzIGJhbm5lZFxuXG5PdG95YSBBbGxvd2VkOlxuXG5HYWdhbWFydSBBbGxvd2VkOlxuLSBQb3BwaW5nIEZhc3QgcmVhY3Rpb25zIGFuZCBHb2RzcGVlZCBzYW1lIHRpbWUgaXMgYmFubmVkXG5cbiIsIiMgUmFyZSBTdHlsZXNcblxuSXNhZ2kgQWxsb3dlZDpcblxuSGlyb3JpIEFsbG93ZWQ6XG4tIFRQIHNob3Qgd2l0aCBsYXNlciBpcyBiYW5uZWRcbi0gQWNlbGVyYXRlIGludG8gLyBpbiBtaW5pYm94IGlzIGJhbm5lZFxuXG5JZ2FndXJpIEFsbG93ZWQ6XG4tIE1hbGljaWE7IG9ubHkgYWxsb3dlZCBpbiB1ciBvd24gaGFsZlxuLSBBaXIgTWFsaWNpYSBhbGxvd2VkIGluIG9wcG9uZW50IGhhbGYgb25seSB3aGVuIHUgaHAVEIGJhbGwgcG9zc2Vzc2lvbiAtIGFzIHBhc3Ncbi0gQXV0byBwYXNzIGludG8gS2Fpc2VyIEltcGFjdCBpcyBiYW5uZWRcbi0gU2tpbGxzIGFyZSBiYW5uZWQgdG8gdXNlIGlmIGEgZmxvdyB0aGF0IHJlZHVjZXMgY29vbGRvd24gaXMgcG9wcGVkXG4iXSwiRmxvdyBSdWxlcyI6WyIjIExpbWl0ZWRcblxuSG9tZWxlc3MgbWFuIEFsbG93ZWQ6XG4taG9tZWxlc3MgbWFuIGlzIHVuYmFubmVkIGJ1dCB1IGNhbiBvbmx5IHNob290IGZyb20gbWlkZmllbClcblxuU3BhZGUgQWxsb3dlZCA6XG4tYnV0IHlvdSBjYW4gb25seSBzaG9vdCBmcm9tIG1pZGZpZWxkIiwiIyBNYXN0ZXIgZmxvd1xuXG5CdXR0ZXJmbHkgRGFuY2VyIEFsbG93ZWQgOlxuLUxhdmluaG8gZmxvdzsgY2FuIHVzZSAyIHBlciB0ZWFtIGJ1dCBvbmx5IDEgaWYgYnVubnkgXG4tZmxvdy9kZXN0cnVjdGl2ZSBpbXB1bHNlcyBpcyBwYWlyZWQgd2l0aCBpdC5cblxuR29kU3BlZWQgQmFubmVkIDpcbiIsIiMgR2VuZXJhdGlvbmFsIGZsb3cgXG5cbkF3YWtlbmVkIEdlbml1cyBBbGxvd2VkIDpcblxuRW1wZXJvciBBbGxvd2VkIDpcbi0gc2hvb3Rpbmcgd2l0aCBFbXBlcm9yIGluIGJveCBkdXJpbmcga2luZ+KAmXMgYXdha2VuaW5nIGlzIGJhbm5lZC5cblxuQWNlIEVhdGVyIEFsbG93ZWQgOlxuXG5UYXJnZXQgTWFuIEFsbG93ZWQgOlxuLUJ1bm55IGZsb3codGFyZ2V0bWFuKTtPbmx5IDEgb2YgaXQgb24gc2FtZSB0ZWFtIG9mZmVuc2l2ZWx5KCBDTSBhbmQgR0sgY2FuIGR1cGUgaXQpLCBiYW5uZWQgd2l0aCBOZWwgUmVvICAsQmFubmVkIHRvIGJlIHBsYXllZCB3aXRoIGFub3RoZXIgZm9yd2FyZCB1c2luZyBkZW1vbiBraW5nLiBcbi1UYXJnZXQgbWFuIGZsb3cgdm9sbGV5IGJhbm5lZCBpbmJveCAobm9ybWFsIG0xIGFsbG93ZWQgaW5ib3ggKVxuIiwiIyBXb3JsZCBDbGFzcyBmbG93IFxuXG5EZW1vbiBraW5nIEFsbG93ZWQgOiBcbi0gT25seSAxIG9mIGl0IG9uIHNhbWUgdGVhbSAob2ZmZW5zaXZlbHksIENNIGFuZCBHSyBjYW5cbi0gZHVwZSBpdCBidXQganVzdCBvbmUgb2YgdGhlbSwgbm90IGJvdGgpLiBCYW5uZWQgdG8gYmUgcGxheWVkIHdpdGggYW5vdGhlciBmb3J3YXJkIHVzaW5nIHRhcmdldCBtYW4gb3IgcHJvZGlneS4gXG5cblNpbmd1bGFyaXR5IEFsbG93ZWQgOlxuLSBvbmx5IDEgcGVyIHRlYW0uXG5cbkNvbnRyYXJpYW4gQWxsb3dlZCA6XG5cbkRlc3RydWN0aXZlIGxtcHVsc2VzIChQcm9kaWd5KSBBbGxvd2VkIDpcbi0gU2hvdCBiYW5uZWQgaW5ib3guIE9ubHkgMSBvZiBpdCBvbiBzYW1lIHRlYW0gKG9mZmVuc2l2ZWx5LCBDTSBhbmQgR0sgY2FuIGR1cGUgaXQpXG4tIGhvb3Rpbmcgd2l0aCBkZXN0cnVjdGl2ZSBpbXB1bHNlcyBpbiBib3ggZHVyaW5nIGtpbmfigJlzIGF3YWtlbmluZyBpcyBiYW5uZWQuXG4iLCIjIE15dGhpYyBcblxuU25ha2UgQWxsb3dlZCA6XG5cbk1hZ2ljaWFuIEZsb3cgQWxsb3dlZCA6XG5cbk1hc3RlciBPZiBBbGwgVHJhZGVzIEFsbG93ZWQgOlxuYW4gYmUgdXNlZCBieSBhbnlvbmUgb24gdGVhbS4gKE1heCBsaW1pdDogbm9uKVxuXG5LaW5ncyBBdXRob3JpdHkgQWxsb3dlZCA6XG4tIHNob290aW5nIHdpdGggS2luZ3MgQXV0aG9yaXR5IGluIGJveCBkdXJpbmcga2luZ+KAmXMgYXdha2VuaW5nIGlzIEJhbm5lZFxuIiwiIyBMZWdlbmRhcnkgXG5cbkNyb3cgQWxsb3dlZCA6XG4tIHNob290aW5nIHdpdGggY3JvdyBpbiBib3ggZHVyaW5nIGtpbmfigJlzIGF3YWtlbmluZyBpcyBiYW5uZWQuXG5cbkRyaWJibGVyIEFsbG93ZWQgOlxuLSBzaG9vdGluZyB3aXRoIGRyaWJibGVyIGluIGJveCBkdXJpbmcga2luZ+KAmXMgYXdha2VuaW5nIGlzIGJhbm5lZC5cblxuVHJhcCAoRmxvdykgQWxsb3dlZCA6XG5cbkRlbW9uIFdpbmdzIEFsbG93ZWQ6XG5cbldpbGQgQ2FyZCBBbGxvd2VkIDogXG5cbiIsIiMgRXBpYyBcblxuR2VuaXVzIEFsbG93ZWQgOiBcblxuTW9uc3RlciBBbGxvd2VkIDogXG5cbkljZSBBbGxvd2VkIDogXG5cblN0ZWFsdGggQWxsb3dlZCA6XG4iLCIjIFJhcmVcblxuTGlnaHRuaW5nIEFsbG93ZWQgOlxuXG5QdXp6bGUgQWxsb3dlZCA6XG5cbkJ1ZGRoYXMgQmxlc3NpbmcgQWxsb3dlZCA6XG4iXSwiR2xvYmFsIE5ld3MgJiBBcmNoaXZlIjpbXX0sImFjY3MiOlt7InVzZXIiOiJyaXBfY2VuayIsInBhc3MiOiIzMjIwMjgwMDQ2NjUwMTAwIiwicm9sZSI6Ik9XTkVSIiwia2V5MmZhIjoiMjIwOTEwIiwiYXZhdGFyIjoiaHR0cHM6Ly9pLnBpbmltZy5jb20vNzM2eC9hYS80Yy80NC9hYTRjNDRiYzEwZTdjMDU1MTgxMGQyMGMxMjJjNThiNy5qcGcifSx7InVzZXIiOiJ4bTIwMDJhbiIsInBhc3MiOiIzMjIwMjgwMCIsInJvbGUiOiJNRU1CRVIiLCJrZXkyZmEiOm51bGwsIm5pY2siOiJ4bTIwMDJhbiIsImF2YXRhciI6IiIsInBlcm0iOiJub25lIn1dLCJyYW5rcyI6W3sibmFtZSI6IkFETUlOIiwicGVybSI6ImFsbCJ9LHsibmFtZSI6Ik1FTUJFUiIsInBlcm0iOiJub25lIn0seyJuYW1lIjoiRmxvdyBydWxlcyBlZGl0b3IiLCJwZXJtIjoiRmxvdyBSdWxlcyJ9LHsibmFtZSI6IlN0eWxlIHJ1bGVzIGVkaXRvciIsInBlcm0iOiJTdHlsZSBSdWxlcyJ9LHsibmFtZSI6IlJhbmtlZCBydWxlcyBlZGl0b3IiLCJwZXJtIjoiUmFua2VkIFJ1bGVzIn0seyJuYW1lIjoiRmxvdyBFZGl0b3IiLCJwZXJtIjoiRmxvdyBSdWxlcyJ9XX0=";

        // --- AUTH & CORE FUNCTIONS ---
        function processAuth() {
            const u = document.getElementById('auth-user').value;
            const p = document.getElementById('auth-pass').value;

            if(authMode === 'login') {
                const acc = accounts.find(a => a.user === u && a.pass === p);
                if(acc) {
                    if(acc.key2fa) {
                        tempUser = acc;
                        document.getElementById('auth-main-fields').style.display = 'none';
                        document.getElementById('auth-2fa-fields').style.display = 'block';
                    } else { completeLogin(acc); }
                } else { notify("Invalid Credentials", "error"); }
            } else {
                if(accounts.find(a => a.user === u)) return notify("User exists", "error");
                const newAcc = { user: u, pass: p, role: "MEMBER", avatar: "", nick: u, perm: "none" };
                accounts.push(newAcc);
                localStorage.setItem('sys_accounts', JSON.stringify(accounts));
                completeLogin(newAcc);
            }
        }

        function completeLogin(acc) {
            currentUser = acc;
            
            // --- AUTO INJECTION LOGIC ---
            // This part forces your F7 data into the user's local storage upon login
            try {
                const decoded = JSON.parse(decodeURIComponent(escape(atob(INJECTION_KEY))));
                localStorage.setItem('joud_core', JSON.stringify(decoded.db));
                joudDB = decoded.db; // Sync current session
                notify("System Synchronized", "success");
            } catch(e) { console.error("Sync Error", e); }
            
            updateProfileUI();
            closeAuthModal();
            notify("Welcome " + (acc.nick || acc.user), "success");
        }

        function updateProfileUI() {
            document.getElementById('auth-section').style.display = 'none';
            document.getElementById('user-profile').style.display = 'flex';
            document.getElementById('user-nick').innerText = currentUser.nick || currentUser.user;
            document.getElementById('user-avatar').src = currentUser.avatar || 'https://via.placeholder.com/35';
        }

        function logout() { location.reload(); }

        function openPage(cat) {
            activeCategory = cat;
            document.getElementById('main-menu').style.display = 'none';
            document.getElementById('page-view').style.display = 'block';
            document.getElementById('back-nav-btn').style.visibility = 'visible';
            document.getElementById('page-title').innerText = cat;
            document.getElementById('admin-panel').style.display = hasPerm(cat) ? 'block' : 'none';
            renderEntries();
        }

        function closePage() { 
            document.getElementById('main-menu').style.display = 'flex'; 
            document.getElementById('page-view').style.display = 'none'; 
            document.getElementById('back-nav-btn').style.visibility = 'hidden'; 
            dragMode = false;
        }

        function hasPerm(cat) {
            if(!currentUser) return false;
            if(currentUser.role === 'OWNER' || currentUser.role === 'ADMIN') return true;
            return currentUser.perm === cat;
        }

        function saveData() {
            const input = document.getElementById('text-input');
            if(!input.value.trim()) return;
            if(editIndex === -1) joudDB[activeCategory].push(input.value);
            else { joudDB[activeCategory][editIndex] = input.value; editIndex = -1; }
            localStorage.setItem('joud_core', JSON.stringify(joudDB));
            input.value = ""; renderEntries();
            notify("Database Updated", "success");
        }

        function renderEntries() {
            const container = document.getElementById('data-list');
            const searchVal = document.getElementById('rule-search').value.toLowerCase();
            const data = joudDB[activeCategory] || [];
            container.innerHTML = "";
            
            data.forEach((item, index) => {
                if(searchVal && !item.toLowerCase().includes(searchVal)) return;

                let lines = item.split('\n');
                let title = lines[0].startsWith('#') ? lines[0].replace('#', '') : 'Rule #' + (index + 1);
                const canEdit = hasPerm(activeCategory);

                const entry = document.createElement('div');
                entry.className = `entry-item ${dragMode ? 'can-drag' : ''}`;
                entry.setAttribute('data-index', index);

                entry.innerHTML = `
                    <div class="entry-header">
                        <span onclick="toggleContent(event, ${index})">${title}</span>
                        <span style="color:${dragMode ? 'var(--blue)' : '#333'};">≡</span>
                    </div>
                    <div class="entry-content" id="content-${index}">
                        <div>${lines.map(l => formatLine(l)).join('')}</div>
                        <div class="admin-controls" style="display: ${canEdit ? 'flex' : 'none'}; gap:10px; margin-top:15px;">
                            <button class="btn" style="padding:5px; font-size:12px; width:60px" onclick="startEdit(${index})">Edit</button>
                            <button class="btn" style="padding:5px; font-size:12px; width:60px; background:var(--red)" onclick="deleteEntry(${index})">Delete</button>
                        </div>
                    </div>
                `;
                container.appendChild(entry);
            });
            container.innerHTML += `<div class="bl-footer">Blue Lock</div>`;
        }

        function formatLine(line) {
            if (!line.trim()) return '<br>';
            const isImage = line.match(/\.(jpeg|jpg|gif|png|webp)/i) || line.includes('ibb.co') || line.includes('discordapp.com');
            if(isImage) return `<img src="${line.trim()}" class="rule-img" onerror="this.style.display='none'">`;
            
            let res = line.replace(/\((.*?)\)/g, '<span class="bracket-text">($1)</span>')
                          .replace(/\bBanned\b/gi, '<span class="word-banned">Banned</span>')
                          .replace(/\bAllowed\b/gi, '<span class="word-allowed">Allowed</span>');
            
            if (line.includes(':')) return `<span class="header-type">${res}</span>`;
            if (line.trim().startsWith('-')) return `<span class="dash-type">${res}</span>`;
            return `<div>${res}</div>`;
        }

        function toggleContent(e, id) { 
            const c = document.getElementById('content-' + id); 
            if(c) c.style.display = (c.style.display !== 'block') ? 'block' : 'none'; 
        }

        function notify(msg, type) {
            const n = document.getElementById('notification');
            n.innerText = msg;
            n.style.background = type === 'success' ? '#004400' : '#440000';
            n.style.display = 'block';
            setTimeout(() => n.style.display = 'none', 3000);
        }

        // --- KEYBOARD SHORTCUTS ---
        window.addEventListener('keydown', e => { 
            if(currentUser?.role === "OWNER" || currentUser?.role === "ADMIN") {
                if(e.key === "F7") { e.preventDefault(); document.getElementById('backup-modal').style.display = 'flex'; generateBackup(); }
                if(e.key === "F4") { e.preventDefault(); dragMode = !dragMode; document.getElementById('reorder-overlay').style.display = dragMode ? 'flex' : 'none'; if(dragMode) renderReorderList(); }
            }
        });

        // Other functions (generateBackup, restoreBackup, renderReorderList etc. from your previous parts)
        function generateBackup() {
            const data = { db: joudDB, accs: accounts, ranks: customRanks };
            const key = btoa(unescape(encodeURIComponent(JSON.stringify(data))));
            document.getElementById('backup-key-box').value = key;
        }

        function restoreBackup() {
            const key = document.getElementById('backup-key-box').value.trim();
            try {
                const decoded = JSON.parse(decodeURIComponent(escape(atob(key))));
                if(confirm("Overwrite current data?")) {
                    localStorage.setItem('joud_core', JSON.stringify(decoded.db));
                    localStorage.setItem('sys_accounts', JSON.stringify(decoded.accs));
                    location.reload();
                }
            } catch(e) { notify("Invalid Key", "error"); }
        }

        function openAuthModal(m) { authMode = m; document.getElementById('auth-modal').style.display = 'flex'; }
        function closeAuthModal() { document.getElementById('auth-modal').style.display = 'none'; }
        function toggleProfileDropdown(e) { e.stopPropagation(); const m = document.getElementById('profile-dropdown-menu'); m.style.display = m.style.display === 'block' ? 'none' : 'block'; }
        document.addEventListener('click', () => { if(document.getElementById('profile-dropdown-menu')) document.getElementById('profile-dropdown-menu').style.display = 'none'; });

    </script>
</body>
</html>
