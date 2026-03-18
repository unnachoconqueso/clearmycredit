<!DOCTYPE html>  
<html>  
<head>  
  <title>ClearMyCredit</title>  
</head>  
<body>  
  
<h1>ClearMyCredit</h1>  
<p>Select your dispute reason:</p>  
  
<select id="reason">  
  <option value="Account not mine">Account not mine</option>  
  <option value="Incorrect late payment">Incorrect late payment</option>  
  <option value="Paid debt still showing">Paid debt still showing</option>  
  <option value="Duplicate account">Duplicate account</option>  
  <option value="Identity theft">Identity theft</option>  
</select>  
  
<br><br>  
  
<button onclick="goToForm()">Generate My Letter</button>  
  
<script>  
function goToForm() {  
  const reason = document.getElementById("reason").value;  
  localStorage.setItem("selectedReason", reason);  
  window.location.href = "form.html";  
}  
</script>  
  
</body>  
</html>  
