<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>UK M1 tax estimation</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            max-width: 600px;
            margin: 50px auto;
            padding: 30px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            box-sizing: border-box;
        }
        
        .container {
            background: white;
            padding: 40px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
        }
        
        h1 {
            color: #333;
            text-align: center;
            margin-bottom: 30px;
            font-size: 2.2em;
        }
        
        .input-section {
            margin-bottom: 30px;
        }
        
        label {
            display: block;
            margin-bottom: 8px;
            font-weight: 600;
            color: #555;
            font-size: 1.1em;
        }
        
        input[type="number"] {
            width: 100%;
            padding: 15px;
            border: 2px solid #ddd;
            border-radius: 8px;
            font-size: 1.1em;
            box-sizing: border-box;
            transition: border-color 0.3s ease;
        }
        
        input[type="number"]:focus {
            outline: none;
            border-color: #667eea;
            box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
        }
        
        .results-section {
            margin-top: 30px;
        }
        
        .single-result {
            display: flex;
            justify-content: center;
            margin-top: 20px;
        }
        
        .single-result .result-item {
            background: #f8f9ff;
            padding: 30px;
            border-radius: 10px;
            border-left: 4px solid #667eea;
            transition: transform 0.2s ease;
            min-width: 200px;
            text-align: center;
        }
        
        .result-item {
            background: #f8f9ff;
            padding: 20px;
            border-radius: 10px;
            border-left: 4px solid #667eea;
            transition: transform 0.2s ease;
        }
        
        .single-result .result-item:hover {
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
        }
        
        .result-label {
            font-weight: 600;
            color: #555;
            margin-bottom: 8px;
            font-size: 0.9em;
        }
        
        .result-value {
            font-size: 1.5em;
            font-weight: 700;
            color: #667eea;
            font-family: 'Courier New', monospace;
        }
        
        .description {
            text-align: center;
            color: #666;
            margin-bottom: 20px;
            font-size: 1.05em;
            line-height: 1.5;
        }
        
            @media (max-width: 480px) {
            body {
                padding: 15px;
            }
            
            .container {
                padding: 25px;
            }
            
            h1 {
                font-size: 1.8em;
            }
            
            .single-result .result-item {
                min-width: auto;
                width: 100%;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>UK M1 tax estimation</h1>
        <p class="description">
            Enter your monthly EC pension amount in GBP to calculate the UK income tax due.
        </p>
        
        <div class="input-section">
            <label for="input1">Monthly EC pension in GBP:</label>
            <input type="number" id="input1" placeholder="Enter monthly pension amount..." step="0.01" min="0">
        </div>
        
        <div class="results-section">
            <h3>Monthly UK Income Tax:</h3>
            <div class="single-result">
                <div class="result-item">
                    <div class="result-label">Tax Due:</div>
                    <div class="result-value" id="taxResult">£0.00</div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // Get references to the input and result elements
        const input1 = document.getElementById('input1');
        const taxResult = document.getElementById('taxResult');
        
        // UK Monthly Tax Band Thresholds (annual amounts ÷ 12)
        const STANDARD_PERSONAL_ALLOWANCE = 1047.50;  // £12,570 ÷ 12
        const BASIC_RATE_THRESHOLD = 4189.17; // £50,270 ÷ 12
        const HIGHER_RATE_THRESHOLD = 10428.33; // £125,140 ÷ 12
        const ALLOWANCE_REDUCTION_THRESHOLD = 8333.33; // £100,000 ÷ 12
        
        // Tax rates
        const BASIC_RATE = 0.20;    // 20%
        const HIGHER_RATE = 0.40;   // 40%
        const ADDITIONAL_RATE = 0.45; // 45%
        
        // Function to calculate adjusted personal allowance based on income
        function calculateAdjustedPersonalAllowance(monthlyIncome) {
            // If income is £100k or less annually (£8,333.33 monthly), full allowance applies
            if (monthlyIncome <= ALLOWANCE_REDUCTION_THRESHOLD) {
                return STANDARD_PERSONAL_ALLOWANCE;
            }
            
            // If income is £125,140 or more annually (£10,428.33 monthly), no allowance
            if (monthlyIncome >= HIGHER_RATE_THRESHOLD) {
                return 0;
            }
            
            // Calculate reduction: £1 reduction for every £2 over £100k annually
            // Monthly equivalent: £1 reduction for every £2 over £8,333.33 monthly
            const excessIncome = monthlyIncome - ALLOWANCE_REDUCTION_THRESHOLD;
            const allowanceReduction = excessIncome / 2; // £1 reduction for every £2 excess
            
            // Calculate adjusted allowance
            const adjustedAllowance = STANDARD_PERSONAL_ALLOWANCE - allowanceReduction;
            
            // Personal allowance cannot go below zero (but this should be caught by the check above)
            return Math.max(0, adjustedAllowance);
        }
        
        // Function to calculate UK monthly income tax with allowance tapering
        function calculateMonthlyTax(monthlyIncome) {
            // Get the adjusted personal allowance
            const personalAllowance = calculateAdjustedPersonalAllowance(monthlyIncome);
            
            // Calculate adjusted thresholds (they move down as allowance reduces)
            const allowanceReduction = STANDARD_PERSONAL_ALLOWANCE - personalAllowance;
            const adjustedBasicRateThreshold = BASIC_RATE_THRESHOLD - allowanceReduction;
            const adjustedHigherRateThreshold = HIGHER_RATE_THRESHOLD - allowanceReduction;
            
            if (monthlyIncome <= personalAllowance) {
                return 0;
            }
            
            let tax = 0;
            
            // Basic rate band
            if (monthlyIncome > personalAllowance) {
                const basicRateTaxableIncome = Math.min(monthlyIncome - personalAllowance, 
                                                       Math.max(0, adjustedBasicRateThreshold - personalAllowance));
                tax += basicRateTaxableIncome * BASIC_RATE;
            }
            
            // Higher rate band
            if (monthlyIncome > adjustedBasicRateThreshold && adjustedBasicRateThreshold > personalAllowance) {
                const higherRateTaxableIncome = Math.min(monthlyIncome - adjustedBasicRateThreshold,
                                                        Math.max(0, adjustedHigherRateThreshold - adjustedBasicRateThreshold));
                tax += higherRateTaxableIncome * HIGHER_RATE;
            }
            
            // Additional rate band
            if (monthlyIncome > adjustedHigherRateThreshold && adjustedHigherRateThreshold > personalAllowance) {
                const additionalRateTaxableIncome = monthlyIncome - adjustedHigherRateThreshold;
                tax += additionalRateTaxableIncome * ADDITIONAL_RATE;
            }
            
            return tax;
        }
        
        // Function to format currency
        function formatCurrency(amount) {
            return '£' + amount.toFixed(2);
        }
        
        // Function to perform calculations and update results
        function calculateAndDisplay() {
            // Get the input value
            const monthlyIncome = parseFloat(input1.value);
            
            // Check if the input is a valid number
            if (isNaN(monthlyIncome) || monthlyIncome < 0) {
                taxResult.textContent = '£0.00';
                return;
            }
            
            // Calculate monthly tax
            const monthlyTax = calculateMonthlyTax(monthlyIncome);
            
            // Display the result
            taxResult.textContent = formatCurrency(monthlyTax);
        }
        
        // Add event listeners to trigger calculation when input changes
        input1.addEventListener('input', calculateAndDisplay);
        input1.addEventListener('change', calculateAndDisplay);
        input1.addEventListener('keyup', calculateAndDisplay);
        
        // Initial call to set up the display
        calculateAndDisplay();
    </script>
</body>
</html>
