# index.html  
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <meta name="viewport" content="width=device-width, initial-scale=1.0">  
    <title>Simple Integration Calculator</title>  
    <script src="https://cdnjs.cloudflare.com/ajax/libs/mathjs/11.8.0/math.js"></script>  
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>  
    <style>  
        body {  
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;  
            background-color: #f5f7fb;  
            margin: 0;  
            padding: 20px;  
            display: flex;  
            justify-content: center;  
        }  
        .container {  
            background-color: #ffffff;  
            padding: 30px;  
            border-radius: 12px;  
            box-shadow: 0 4px 15px rgba(0,0,0,0.05);  
            max-width: 900px;  
            width: 100%;  
            display: grid;  
            grid-template-columns: 1fr 2fr;  
            gap: 30px;  
        }  
        @media (max-width: 768px) {  
            .container { grid-template-columns: 1fr; }  
        }  
        .control-panel {  
            display: flex;  
            flex-direction: column;  
            gap: 15px;  
        }  
        h2 {  
            margin-top: 0;  
            color: #333;  
        }  
        .input-group {  
            display: flex;  
            flex-direction: column;  
            gap: 5px;  
        }  
        label {  
            font-weight: 600;  
            color: #666;  
            font-size: 0.9rem;  
        }  
        input {  
            padding: 10px;  
            border: 1px solid #ccc;  
            border-radius: 6px;  
            font-size: 1rem;  
        }  
        input:focus {  
            outline: none;  
            border-color: #3b82f6;  
            box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);  
        }  
        button {  
            padding: 12px;  
            background-color: #3b82f6;  
            color: white;  
            border: none;  
            border-radius: 6px;  
            font-size: 1rem;  
            font-weight: bold;  
            cursor: pointer;  
            transition: background 0.2s;  
        }  
        button:hover {  
            background-color: #2563eb;  
        }  
        .result-box {  
            margin-top: 15px;  
            padding: 15px;  
            background-color: #eff6ff;  
            border-left: 5px solid #3b82f6;  
            border-radius: 4px;  
        }  
        .result-title {  
            font-size: 0.85rem;  
            color: #1e40af;  
            text-transform: uppercase;  
            letter-spacing: 0.5px;  
            margin-bottom: 5px;  
        }  
        .result-value {  
            font-size: 1.5rem;  
            font-weight: bold;  
            color: #1d4ed8;  
        }  
        .error {  
            color: #dc2626;  
            font-size: 0.9rem;  
            margin-top: 5px;  
            display: none;  
        }  
        .graph-panel {  
            position: relative;  
            min-height: 300px;  
        }  
    </style>  
</head>  
<body>  
  
<div class="container">  
    <div class="control-panel">  
        <h2>Definite Integral</h2>  
          
        <div class="input-group">  
            <label for="functionInput">Function f(x)</label>  
            <input type="text" id="functionInput" value="x^2 - 2x" placeholder="e.g., x^2, sin(x), exp(x)">  
        </div>  
  
        <div class="input-group">  
            <label for="lowerBound">Lower Bound (a)</label>  
            <input type="number" id="lowerBound" value="0" step="any">  
        </div>  
  
        <div class="input-group">  
            <label for="upperBound">Upper Bound (b)</label>  
            <input type="number" id="upperBound" value="3" step="any">  
        </div>  
  
        <button onclick="calculateAndGraph()">Calculate & Plot</button>  
          
        <div id="errorMessage" class="error"></div>  
  
        <div class="result-box">  
            <div class="result-title">∫ f(x) dx from a to b</div>  
            <div id="resultValue" class="result-value">0.00</div>  
        </div>  
    </div>  
  
    <div class="graph-panel">  
        <canvas id="graphCanvas"></canvas>  
    </div>  
</div>  
  
<script>  
    let myChart = null;  
  
    function calculateAndGraph() {  
        const errorDiv = document.getElementById('errorMessage');  
        errorDiv.style.display = 'none';  
          
        const fStr = document.getElementById('functionInput').value;  
        const a = parseFloat(document.getElementById('lowerBound').value);  
        const b = parseFloat(document.getElementById('upperBound').value);  
  
        if (isNaN(a) || isNaN(b)) {  
            showError("Please enter valid numbers for bounds.");  
            return;  
        }  
  
        try {  
            // Compile function using Math.js  
            const expr = math.compile(fStr);  
              
            // 1. Calculate the Definite Integral using Simpson's Rule  
            const steps = 1000; // higher steps = higher accuracy  
            const h = (b - a) / steps;  
            let integralSum = expr.evaluate({x: a}) + expr.evaluate({x: b});  
  
            for (let i = 1; i < steps; i++) {  
                const x = a + i * h;  
                const coef = (i % 2 === 0) ? 2 : 4;  
                integralSum += coef * expr.evaluate({x: x});  
            }  
            const result = (h / 3) * integralSum;  
              
            // Display Result  
            document.getElementById('resultValue').innerText = isNaN(result) ? "Error" : result.toFixed(5);  
  
            // 2. Generate data for Plotting  
            // We expand the view slightly outside the bounds [a, b] for context  
            const padding = Math.max(1, Math.abs(b - a) * 0.5);  
            const viewMin = a - padding;  
            const viewMax = b + padding;  
              
            const labels = [];  
            const mainData = [];  
            const integralFillData = [];  
  
            const totalPoints = 200;  
            const stepSize = (viewMax - viewMin) / totalPoints;  
  
            for (let i = 0; i <= totalPoints; i++) {  
                const x = viewMin + i * stepSize;  
                let y;  
                try {  
                    y = expr.evaluate({x: x});  
                } catch(e) {  
                    y = null;  
                }  
  
                labels.push(x.toFixed(2));  
                mainData.push({x: x, y: y});  
  
                // Fill the area under the curve ONLY between a and b  
                if (x >= a && x <= b) {  
                    integralFillData.push({x: x, y: y});  
                }  
            }  
  
            // Update Chart.js Graphic  
            renderChart(mainData, integralFillData, a, b);  
  
        } catch (error) {  
            showError("Invalid function expression. Use standard math syntax (e.g., x^2, sin(x)).");  
            console.error(error);  
        }  
    }  
  
    function showError(msg) {  
        const errorDiv = document.getElementById('errorMessage');  
        errorDiv.innerText = msg;  
        errorDiv.style.display = 'block';  
        document.getElementById('resultValue').innerText = "—";  
    }  
  
    function renderChart(mainLine, filledArea, a, b) {  
        const ctx = document.getElementById('graphCanvas').getContext('2d');  
  
        if (myChart) {  
            myChart.destroy();  
        }  
  
        myChart = new Chart(ctx, {  
            type: 'line',  
            data: {  
                datasets: [  
                    {  
                        label: 'f(x)',  
                        data: mainLine,  
                        borderColor: '#2563eb',  
                        borderWidth: 2,  
                        pointRadius: 0,  
                        fill: false,  
                        order: 1  
                    },  
                    {  
                        label: `Area (from ${a} to ${b})`,  
                        data: filledArea,  
                        backgroundColor: 'rgba(59, 130, 246, 0.3)',  
                        borderColor: 'transparent',  
                        pointRadius: 0,  
                        fill: 'origin', // Fills area down to the x-axis (y=0)  
                        order: 2  
                    }  
                ]  
            },  
            options: {  
                responsive: true,  
                maintainAspectRatio: false,  
                scales: {  
                    x: {  
                        type: 'linear',  
                        position: 'bottom',  
                        title: { display: true, text: 'x' }  
                    },  
                    y: {  
                        title: { display: true, text: 'y' }  
                    }  
                },  
                plugins: {  
                    legend: {  
                        labels: {  
                            filter: function(item) {  
                                // Clean up the legend view  
                                return item.text !== undefined;  
                            }  
                        }  
                    }  
                }  
            }  
        });  
    }  
  
    // Run once on load  
    window.onload = calculateAndGraph;  
</script>  
</body>  
</html>  
