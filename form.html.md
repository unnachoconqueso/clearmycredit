<!DOCTYPE html>  
<html>  
<head>  
  <title>Fill Your Info</title>  
</head>  
<body>  
  
<h2>Fill Your Information</h2>  
  
<input type="text" id="userName" placeholder="Your Name"><br><br>  
<input type="text" id="userAddress" placeholder="Your Address"><br><br>  
<input type="text" id="accountNumber" placeholder="Account Number"><br><br>  
<input type="text" id="bankName" placeholder="Bank / Creditor Name"><br><br>  
<input type="text" id="creditBureau" placeholder="Credit Bureau"><br><br>  
<input type="date" id="reportDate"><br><br>  
  
<button onclick="generateFinalLetter()">Generate My Letter</button>  
  
<script>  
function generateFinalLetter() {  
  
  const name = document.getElementById("userName").value;  
  const address = document.getElementById("userAddress").value;  
  const account = document.getElementById("accountNumber").value;  
  const bank = document.getElementById("bankName").value;  
  const bureau = document.getElementById("creditBureau").value;  
  const date = document.getElementById("reportDate").value;  
  const reason = localStorage.getItem("selectedReason");  
  
  const letter = `To ${bureau},  
  
I am writing to dispute the following information in my credit report:  
  
Account: ${account}  
Creditor: ${bank}  
Date of Last Reported Status: ${date}  
  
The reason for this dispute is: ${reason}  
  
Please investigate and remove or correct this item.  
  
Sincerely,  
${name}  
${address}`;  
  
  localStorage.setItem("finalLetter", letter);  
  
  window.location.href = "result.html";  
}  
</script>  
  
</body>  
</html>  
