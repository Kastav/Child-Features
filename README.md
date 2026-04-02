<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Ultimate AI Parent & Child Predictor</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<style>
body { font-family: Arial; margin:20px; background:#f9f9f9; color:#333;}
h1 { color:#0077cc;}
label { display:block; margin-top:10px; font-weight:bold; }
input, select, button { margin-top:5px; padding:5px; width:250px; }
button { background:#0077cc; color:white; border:none; cursor:pointer; }
button:hover { background:#005fa3; }
#result { margin-top:20px; padding:15px; background:#e2f0ff; border-left:5px solid #0077cc; white-space: pre-wrap;}
img {max-width:150px; margin-top:10px;}
#childPreview {border:1px solid #0077cc; margin-top:10px; max-width:250px;}
</style>
</head>
<body>

<h1>Ultimate AI Parent & Child Predictor</h1>
<p>Observe a person to predict their parents' traits, or upload two parents to predict a future child.</p>

<h2>Observe Any Person</h2>
<label>Person Photo</label>
<input type="file" id="personPhoto" accept="image/*">
<img id="personPreview" src="" alt="Person Preview">

<h2>Optional: Future Child Prediction</h2>
<label>Parent 1 Photo</label>
<input type="file" id="parent1" accept="image/*">
<img id="preview1" src="" alt="Parent 1 Preview">

<label>Parent 2 Photo</label>
<input type="file" id="parent2" accept="image/*">
<img id="preview2" src="" alt="Parent 2 Preview">

<label>Desired Child Gender (optional)</label>
<select id="childGender">
  <option value="">Any</option>
  <option value="male">Male</option>
  <option value="female">Female</option>
</select>

<label>API Key (for cloud AI model, optional if using self-hosted backend)</label>
<input type="text" id="apiKey" placeholder="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">

<button onclick="analyzePerson()">Analyze / Predict</button>
<button onclick="downloadPDF()">Download PDF Report</button>

<div id="result"></div>
<img id="childPreview" src="" alt="Child Face Preview">

<script>
const { jsPDF } = window.jspdf;
let lastReport = '';

const personPreview = document.getElementById('personPreview');
const preview1 = document.getElementById('preview1');
const preview2 = document.getElementById('preview2');
const childPreview = document.getElementById('childPreview');

document.getElementById('personPhoto').addEventListener('change', e => personPreview.src = URL.createObjectURL(e.target.files[0]));
document.getElementById('parent1').addEventListener('change', e => preview1.src = URL.createObjectURL(e.target.files[0]));
document.getElementById('parent2').addEventListener('change', e => preview2.src = URL.createObjectURL(e.target.files[0]));

async function analyzePerson() {
  const personImg = personPreview;
  const p1 = preview1;
  const p2 = preview2;
  const gender = document.getElementById("childGender").value;
  const apiKey = document.getElementById("apiKey").value;

  if(!personImg.src && (!p1.src || !p2.src)){
    alert('Upload at least one person or both parents for child prediction.');
    return;
  }
  
  document.getElementById('result').innerHTML = 'Analyzing...';

  // ----- Observe Person -> Parent Details -----
  if(personImg.src){
    // Example logic, replace with ML/AI model for production
    const personGender = gender || "male";
    const predictedFatherHeight = personGender==="male"?"~160-175 cm":"~150-165 cm";
    const predictedMotherHeight = personGender==="male"?"~150-165 cm":"~155-165 cm";
    const predictedFatherBody = personGender==="male"?"Father-like (~60%)":"Mix (~50%)";
    const predictedMotherBody = personGender==="female"?"Mother-like (~60%)":"Mix (~50%)";
    const predictedSkin = ["White","Brown","Dark"][Math.floor(Math.random()*3)];
    const facialShape = ["Oval","Round","Square"][Math.floor(Math.random()*3)];

    lastReport = `
Observed Person - Parent Prediction
------------------------------------
Predicted Father's Height: ${predictedFatherHeight}
Predicted Father's Body Structure: ${predictedFatherBody}
Predicted Mother's Height: ${predictedMotherHeight}
Predicted Mother's Body Structure: ${predictedMotherBody}
Predicted Skin Color: ${predictedSkin}
Predicted Facial Shape: ${facialShape}
Special Features: ${personGender==="male"?"Beard possibility from father":"Braid possibility from mother"}
Disclaimer: Predictions are probabilistic. Some traits may resemble grandparents or skipped generations.
`;
    document.getElementById('result').innerText = lastReport;
  }

  // ----- Predict Future Child -----
  if(p1.src && p2.src){
    try{
      // Prepare FormData
      const formData = new FormData();
      formData.append("parent1_photo", document.getElementById("parent1").files[0]);
      formData.append("parent2_photo", document.getElementById("parent2").files[0]);
      formData.append("desired_child_gender", gender);

      // Send to backend
      const headers = {};
      if(apiKey) headers["Authorization"] = `Token ${apiKey}`;

      const response = await fetch("https://YOUR_API_ENDPOINT/predict_child", {
        method: "POST",
        body: formData,
        headers: headers
      });

      if(!response.ok){ throw new Error("Child generation failed"); }
      const data = await response.json();
      childPreview.src = data.child_image_url;

      lastReport += `\n\nFuture Child Prediction
-----------------------------
Predicted Gender: ${gender||'Any'}
Predicted Height: ${data.traits?.height_range || 'N/A'}
Predicted Body Structure: ${data.traits?.body_structure || 'Mix of parents'}
Predicted Skin Color: ${data.traits?.skin_color || 'N/A'}
Predicted Facial Shape: ${data.traits?.facial_shape || 'N/A'}
Predicted Hair Color: ${data.traits?.hair_color || 'N/A'}
Predicted Eye Color: ${data.traits?.eye_color || 'N/A'}
Special Features: ${gender==='male'?'Beard potential (~60%)':'Braid potential (~60%)'}
Disclaimer: Probabilities are approximate and some traits may resemble grandparents or skipped generations.
`;

      document.getElementById('result').innerText = lastReport;

    }catch(err){
      alert(err);
      document.getElementById('result').innerText = "Error generating child face. Check API and key.";
    }
  }
}

// PDF Download
function downloadPDF(){
  if(!lastReport){ alert('Generate prediction first'); return; }
  const doc = new jsPDF();
  const lines = lastReport.split('\n'); let y=10;
  lines.forEach(l => { doc.text(l, 10, y); y += 10; });
  if(childPreview.src){
    doc.addImage(childPreview.src,'PNG',10,y,80,80);
  }
  doc.save('ParentChildPrediction.pdf');
}
</script>

</body>
</html>
