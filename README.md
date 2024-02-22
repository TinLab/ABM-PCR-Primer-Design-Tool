<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ABM-PCR Primer Design Tool for single-base mutation detection</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            background-color: #f4f4f4;
            margin: 0;
            padding: 20px;
            color: #333;
        }
        .container {
            max-width: 600px;
            margin: auto;
            background: white;
            padding: 20px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            border-radius: 8px;
        }
        h1, h2 {
            text-align: center;
        }
        form {
            display: flex;
            flex-direction: column;
            gap: 10px;
        }
        input[type="text"] {
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 16px;
        }
        input[type="submit"] {
            padding: 10px 20px;
            border: none;
            background-color: #5cb85c;
            color: white;
            font-size: 16px;
            cursor: pointer;
            border-radius: 4px;
            transition: background-color 0.3s;
        }
        input[type="submit"]:hover {
            background-color: #4cae4c;
        }
        #result {
            position: relative; 
            margin-top: 20px;
            padding: 10px;
            background-color: #e9ffe9;
            border: 1px dashed #4cae4c;
            border-radius: 4px;
            padding-top: 40px;
        }
        .copyButton {
            position: absolute;
            top: 10px;
            right: 10px;
            cursor: pointer;
            background-color: #4CAF50;
            color: white;
            padding: 5px 10px;
            border: none;
            border-radius: 4px;
        }

        #legend {
            margin-top: 20px;
            padding: 10px;
            background-color: #f9f9f9;
            border: 1px solid #ddd;
            border-radius: 4px;
        }

        .legend-color {
            display: inline-block;
            width: 20px;
            height: 20px;
            border-radius: 4px;
            margin-right: 5px;
        }

        .mutated {
            background-color: red;
        }

        .artificial {
            background-color: blue;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>ABM-PCR Primer Design Tool for single-base mutation detection</h1>
        <form id="dnaForm">
            <label for="dnaInput">Enter ARMS-PCR primer DNA sequence (18-24 bp):</label>
            <input type="text" id="dnaInput" name="dna" pattern="[ATCGatcg]{18,24}" title="DNA sequence should be 18-24 bp long and only contain A, T, C, or G.">
            <div id="mutationOptions">
                <label>Select Mutation Base:</label>
                <input type="radio" id="lastBaseOption" name="mutationOption" value="lastBase" checked>
                <label for="lastBaseOption">Last Base (Default)</label>
                <input type="radio" id="secondLastBaseOption" name="mutationOption" value="secondLastBase">
                <label for="secondLastBaseOption">Second Last Base</label>
            </div>
            <input type="submit" value="Submit" id="submitButton">
        </form>

        <div id="legend">
            <h2>Color Legend:</h2>
            <p><span class="legend-color mutated"></span> Mutated Base (Red)</p>
            <p><span class="legend-color artificial"></span> Artificial Base Mismatch (Blue)</p>
        </div>

        <h2>Suggested Primer Sequences:</h2>
        <div id="result">
            <button class="copyButton" onclick="copySequences()">Copy All Sequences</button>
        </div>




    <script>

var globalFlag = 0; 
		
document.getElementById('dnaForm').onsubmit = function(event) {
    event.preventDefault();
    
    var dnaSequence = document.getElementById('dnaInput').value.toUpperCase();
    var mutationOption = document.querySelector('input[name="mutationOption"]:checked').value;
    
    if(/^[ATCGatcg]+$/.test(dnaSequence)) {
        var resultHTML = '';
        if (mutationOption === 'secondLastBase') {
            
            resultHTML = processDNASequence2(dnaSequence);
			
        } else {
            
            resultHTML = processDNASequence(dnaSequence);

        }
        updateResult(resultHTML);
    } else {
        alert("Please enter a valid DNA sequence containing only A, T, C, or G.");
        updateResult('');
    }
};



		
function updateResult(newContent) {
    var resultContainer = document.getElementById('result');
    var resultContent = document.createElement('div');
    resultContent.innerHTML = newContent;
    
    if (resultContainer.childNodes.length > 1) {
        resultContainer.removeChild(resultContainer.lastChild);
    }
   
    resultContainer.appendChild(resultContent);
}



function processDNASequence(sequence) {
    var n = sequence.length;
    var result = `<p>ARMS-PCR Primer Sequence: ${sequence}</p>` +
                 `<p>Total Number of Bases: ${n}</p>`;

    var lastBase = sequence.charAt(n - 1);
    var lastBaseColored = `<span style="color:red;">${lastBase}</span>`;
    result += `<p>${sequence.slice(0, n - 1) + lastBaseColored}</p>`;

    var sequences = createMisMatches(lastBase, sequence.slice(0, n - 1), lastBaseColored, "blue")
        .concat(createMisMatches(lastBase, sequence.slice(0, n - 2), sequence.charAt(n - 2) + lastBaseColored, "blue"));
	
	if (globalFlag === 0) {
    sequences.forEach(seqObj => {
   
		if (seqObj.flag === 1) {
            var modifiedResult = applyRuleP(seqObj, n);
            if (modifiedResult) {
                result += modifiedResult;
            }
        } else if (seqObj.flag === 0){

	}
        
    });
	}
	
	else if(globalFlag === 1){
		sequences.forEach(seqObj => {
        result += `<p>${seqObj.sequence}</p>`;
    });
	}
    return result;
}

function processDNASequence2(sequence) {
    var n = sequence.length;
    var result = `<p>ARMS-PCR Primer Sequence: ${sequence}</p>` +
                 `<p>Total Number of Bases: ${n}</p>`;
	var lastfive = sequence.slice(-5);
	var lastBase = sequence.charAt(n - 1);
    var lasttwoBase = sequence.charAt(n - 2);
    var lasttwoBaseColored = `<span style="color:red;">${lasttwoBase}</span>`;
    result += `<p>${sequence.slice(0, n - 2) + lasttwoBaseColored + lastBase}</p>`;

    var sequences = createMisMatches2(lastBase, lastfive, lasttwoBase, sequence.slice(0, n - 2), lasttwoBaseColored, "blue")
   
    if (globalFlag === 0) {
      
	var modifiedResult = applyRuleP2(sequences, n);
    if (modifiedResult) {
        result += modifiedResult;
    }
    }
	else if(globalFlag === 1){
		result += `<p>${sequences}</p>`;
	}
	
    return result;

}		
		
function createMisMatches (base, sequence, lastBaseColored, color) {
    var newBase;
	var n = sequence.length;
    var oldBase = sequence.charAt(n - 1);
	var flag = 0;
	
    if (oldBase === 'G') {
        newBase = 'A';
    } else if (oldBase === 'C') {
        newBase = 'T';
	} else if (oldBase === 'A') {
        newBase = 'T';
    } else if (oldBase === 'T') {
        newBase = 'T';
    }

	if (newBase !== oldBase) {
    var newBaseColored = `<span style="color:${color};">${newBase}</span>`;
    var newSequence = sequence.slice(0, -1) + newBaseColored + lastBaseColored;
    var flag = 1;
	}
	else{
		var newSequence = sequence + lastBaseColored;
    	var flag = 0;
	}
	return [{ sequence: newSequence, flag: flag }];
}
		
function createMisMatches2(lastBase, lastfive, base, sequence, lasttwoBaseColored, color) {
    var newBase;
    
    if (base === 'C' || base === 'G') {
        newBase = 'A';
    } else if (base === 'A') {
        newBase = 'T';
	} else if (base === 'T') {
        newBase = 'T';	
    } else if (!lastfive.includes('A') && !lastfive.includes('T')) {
        newBase = 'A';
    }
	
    var newBaseColored = `<span style="color:${color};">${newBase}</span>`;
    var newSequence = sequence.slice(0, -1) + newBaseColored + lasttwoBaseColored + lastBase;
    var flag = 0;
	globalFlag = flag; 
	
    return newSequence;
	
}

function applyRuleP2(newSequence, n) {
 
    var start = 6; // 7th base (0-indexed)
    var end = n - 7; // n-7th base (0-indexed)
    var result = '';
	
    for (var i = end; i >= start; i--) {
        var sequence = newSequence;
		var modifiedSequence = sequence.split('');
        var base = modifiedSequence[i].replace(/<[^>]*>/g, ""); // Remove HTML tags
        var modified = false;

        if (base === 'C') {
            modifiedSequence[i] = `<span style="color:blue;">T</span>`;
            modified = true;
        } else if (base === 'G' ) {
            modifiedSequence[i] = `<span style="color:blue;">A</span>`;
            modified = true;
        }else if (base === 'T' ) {
            modifiedSequence[i] = `<span style="color:blue;">A</span>`;
            modified = true;
        }else if (base === 'A' ) {
            modifiedSequence[i] = `<span style="color:blue;">T</span>`;
            modified = true;
        }

        if (modified) {
            result += `<p>Modified at position ${i}: ${modifiedSequence.join('')}</p>`;
        }
    }

    return result;
}
		
function applyRuleP(sequenceObj, n) {
    var sequence = sequenceObj.sequence;
    var flag = sequenceObj.flag;
    var start = 6; // 7th base (0-indexed)
    var end = n - 7; // n-7th base (0-indexed)
    var result = '';

    for (var i = end; i >= start; i--) {
        var modifiedSequence = sequence.split('');
        var base = modifiedSequence[i].replace(/<[^>]*>/g, ""); // Remove HTML tags
        var modified = false;

        if (base === 'C') {
            modifiedSequence[i] = `<span style="color:blue;">T</span>`;
            modified = true;
        } else if (base === 'G' ) {
            modifiedSequence[i] = `<span style="color:blue;">A</span>`;
            modified = true;
        }else if (base === 'T' ) {
            modifiedSequence[i] = `<span style="color:blue;">A</span>`;
            modified = true;
        }else if (base === 'A' ) {
            modifiedSequence[i] = `<span style="color:blue;">T</span>`;
            modified = true;
        }

        if (modified) {
            result += `<p>Modified at position ${i}: ${modifiedSequence.join('')}</p>`;
        }
    }

    return result;
}		
		
		
function copySequences() {
            var sequences = document.getElementById('result').innerText;
            var textArea = document.createElement('textarea');
            textArea.value = sequences;
            document.body.appendChild(textArea);
            textArea.select();
            document.execCommand('Copy');
            textArea.remove();
            alert('Sequences copied to clipboard!');
        
        }
		
</script>
</body>
</html>
