<!DOCTYPE html>  
<html>  
<head>  
  <title>Your Letter</title>  
</head>  
<body>  
  
<h2>Your Dispute Letter</h2>  
  
<div id="letter" style="white-space: pre-wrap; border:1px solid black; padding:15px;"></div>  
  
<br>  
  
<button onclick="copyText()">Copy Letter</button>  
  
<script>  
const letter = localStorage.getItem("finalLetter");  
document.getElementById("letter").innerText = letter;  
  
function copyText() {  
  navigator.clipboard.writeText(letter);  
  alert("Letter copied!");  
}  
</script>  
  
</body>  
</html>  
