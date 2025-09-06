js
app.use(express.static("public"));
// ===== Health =====
app.get("/api/health", (req, res) => {
  res.send("Micro-Lending API running ✅");
});

app.use(express.static("public"));  // <-- serve frontend

app.listen(4000, () => console.log("Server running"));
//HTML 

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Micro-Lending Platform</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 20px; }
    input, button { margin: 5px; padding: 5px; }
    .loan { border: 1px solid #ccc; padding: 10px; margin: 10px 0; }
  </style>
</head>
<body>
  <h1>Micro-Lending Platform</h1>

  <h2>Register / Login</h2>
  <input id="email" placeholder="Email">
  <input id="password" type="password" placeholder="Password">
  <select id="role">
    <option value="BORROWER">Borrower</option>
    <option value="LENDER">Lender</option>
  </select>
  <button onclick="register()">Register</button>
  <button onclick="login()">Login</button>
  <p id="authStatus"></p>

  <h2>Create Loan (Borrower)</h2>
  <input id="loanTitle" placeholder="Title">
  <input id="loanAmount" type="number" placeholder="Amount">
  <input id="loanDesc" placeholder="Description">
  <button onclick="createLoan()">Post Loan</button>

  <h2>Loans Marketplace</h2>
  <div id="loans"></div>

  <script src="script.js"></script>
</body>
</html>


//PUBLIC/SCRIPT.JS

const API = ""; // same host
let token = null;

async function register() {
  const email = document.getElementById("email").value;
  const password = document.getElementById("password").value;
  const role = document.getElementById("role").value;
  const res = await fetch("/register", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ email, password, role })
  });
  const data = await res.json();
  document.getElementById("authStatus").innerText = JSON.stringify(data);
}

async function login() {
  const email = document.getElementById("email").value;
  const password = document.getElementById("password").value;
  const res = await fetch("/login", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ email, password })
  });
  const data = await res.json();
  token = data.token;
  document.getElementById("authStatus").innerText = "Logged in ✅";
  loadLoans();
}

async function createLoan() {
  if (!token) return alert("Login as BORROWER first");
  const title = document.getElementById("loanTitle").value;
  const amount = Number(document.getElementById("loanAmount").value);
  const description = document.getElementById("loanDesc").value;
  await fetch("/loans", {
    method: "POST",
    headers: { "Content-Type": "application/json", "Authorization": `Bearer ${token}` },
    body: JSON.stringify({ title, amount, description })
  });
  loadLoans();
}

async function fundLoan(id) {
  if (!token) return alert("Login as LENDER first");
  await fetch(`/loans/${id}/fund`, {
    method: "POST",
    headers: { "Content-Type": "application/json", "Authorization": `Bearer ${token}` },
    body: JSON.stringify({ amount: 50 })
  });
  loadLoans();
}

async function repayLoan(id) {
  if (!token) return alert("Login as BORROWER first");
  await fetch(`/loans/${id}/repay`, {
    method: "POST",
    headers: { "Content-Type": "application/json", "Authorization": `Bearer ${token}` },
    body: JSON.stringify({ amount: 50 })
  });
  loadLoans();
}

async function loadLoans() {
  const res = await fetch("/loans");
  const loans = await res.json();
  const div = document.getElementById("loans");
  div.innerHTML = "";
  loans.forEach(l => {
    const d = document.createElement("div");
    d.className = "loan";
    d.innerHTML = `
      <b>${l.title}</b> — $${l.amount} — funded: $${l.funded} — status: ${l.status}
      <br>${l.description}
      <br>
      <button onclick="fundLoan(${l.id})">Fund $50</button>
      <button onclick="repayLoan(${l.id})">Repay $50</button>
    `;
    div.appendChild(d);
  });
}


