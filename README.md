# test2
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>고용유지지원금 모의 계산기</title>
    <style>
        :root { --primary: #2563eb; --secondary: #64748b; --bg: #f8fafc; --white: #ffffff; }
        body { font-family: 'Pretendard', sans-serif; background-color: var(--bg); color: #334155; line-height: 1.6; padding: 20px; }
        .wrapper { max-width: 800px; margin: 0 auto; background: var(--white); padding: 40px; border-radius: 16px; box-shadow: 0 10px 25px rgba(0,0,0,0.05); }
        h1 { text-align: center; color: var(--primary); margin-bottom: 30px; }
        .section { margin-bottom: 30px; padding: 20px; border: 1px solid #e2e8f0; border-radius: 12px; }
        .section-title { font-weight: bold; font-size: 1.1rem; margin-bottom: 15px; display: flex; align-items: center; color: #1e293b; }
        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
        label { display: block; font-size: 0.9rem; margin-bottom: 6px; color: #475569; }
        select, input { width: 100%; padding: 10px; border: 1px solid #cbd5e1; border-radius: 6px; box-sizing: border-box; }
        .info-box { background: #f1f5f9; padding: 15px; border-radius: 8px; font-size: 0.85rem; margin-top: 10px; }
        button { width: 100%; padding: 16px; background: var(--primary); color: white; border: none; border-radius: 8px; font-size: 1.1rem; font-weight: bold; cursor: pointer; margin-top: 20px; }
        #resultArea { margin-top: 30px; padding: 25px; border-radius: 12px; display: none; }
        .pass { background: #ecfdf5; border: 1px solid #10b981; }
        .fail { background: #fef2f2; border: 1px solid #ef4444; }
        .result-val { font-size: 1.8rem; font-weight: 800; color: #059669; }
    </style>
</head>
<body>

<div class="wrapper">
    <h1>💼 고용유지지원금 계산기</h1>

    <div class="section">
        <div class="section-title">1. 기업 유형 및 매출 조건</div>
        <div class="grid">
            <div>
                <label>기업 분류</label>
                <select id="companyType">
                    <option value="priority">우선지원대상기업</option>
                    <option value="large">대규모 기업</option>
                </select>
            </div>
            <div>
                <label>매출 감소율 (%)</label>
                <input type="number" id="revenueDrop" placeholder="예: 20">
            </div>
        </div>
        <div class="info-box">
            * 유급: 15% 이상 감소 시 대상 / 무급: 30% 이상 감소 시 대상
        </div>
    </div>

    <div class="section">
        <div class="section-title">2. 지원 신청 내용</div>
        <div class="grid">
            <div>
                <label>지원금 종류</label>
                <select id="supportType" onchange="toggleInputs()">
                    <option value="paid">유급 고용유지지원금</option>
                    <option value="unpaid">무급 고용유지지원금</option>
                </select>
            </div>
            <div id="extraOption">
                <label>근로시간 단축률 50% 이상인가요?</label>
                <select id="isHalfReduction">
                    <option value="no">아니오</option>
                    <option value="yes">예</option>
                </select>
            </div>
        </div>
    </div>

    <div class="section">
        <div class="section-title">3. 계산 상세 정보</div>
        <div class="grid">
            <div id="wageInputArea">
                <label>1인당 1일 지급액 (유급만 해당)</label>
                <input type="number" id="dailyWage" placeholder="예: 80000">
            </div>
            <div>
                <label>대상 인원 (명)</label>
                <input type="number" id="empCount" placeholder="예: 5">
            </div>
            <div>
                <label>고용유지 기간 (일)</label>
                <input type="number" id="days" max="180" placeholder="최대 180일">
            </div>
        </div>
    </div>

    <button onclick="runCalculator()">분석 및 예상 지원금 계산</button>

    <div id="resultArea">
        <h3 id="resStatus">진단 결과</h3>
        <p id="resMsg"></p>
        <div id="resCalc" style="margin-top:15px;">
            <span style="font-size: 1rem;">최대 예상 지원금:</span><br>
            <span class="result-val" id="totalValue">0원</span>
        </div>
    </div>
</div>

<script>
    function toggleInputs() {
        const type = document.getElementById('supportType').value;
        document.getElementById('wageInputArea').style.display = (type === 'paid') ? 'block' : 'none';
        document.getElementById('extraOption').style.display = (type === 'paid') ? 'block' : 'none';
    }

    function runCalculator() {
        // 데이터 가져오기
        const companyType = document.getElementById('companyType').value;
        const revenueDrop = parseFloat(document.getElementById('revenueDrop').value) || 0;
        const supportType = document.getElementById('supportType').value;
        const isHalfReduction = document.getElementById('isHalfReduction').value === 'yes';
        const dailyWage = parseFloat(document.getElementById('dailyWage').value) || 0;
        const empCount = parseInt(document.getElementById('empCount').value) || 0;
        const days = Math.min(parseInt(document.getElementById('days').value) || 0, 180);

        const resArea = document.getElementById('resultArea');
        const resStatus = document.getElementById('resStatus');
        const resMsg = document.getElementById('resMsg');
        const totalValue = document.getElementById('totalValue');

        // 1. 자격 진단 (매출 요건)
        let isEligible = false;
        let minDrop = (supportType === 'paid') ? 15 : 30;
        
        if (revenueDrop >= minDrop) isEligible = true;

        resArea.style.display = 'block';

        if (!isEligible) {
            resArea.className = 'fail';
            resStatus.innerText = "⚠️ 지원 부적격";
            resMsg.innerText = `${supportType === 'paid' ? '유급' : '무급'} 지원금 요건인 매출 감소율 ${minDrop}%에 미달합니다.`;
            totalValue.innerText = "0원";
            return;
        }

        // 2. 지원금 계산
        let dailySubsidy = 0;
        const LIMIT = 66000;

        if (supportType === 'paid') {
            // 유급: 지급액의 2/3 (우선지원) 또는 1/2 (대규모)
            let rate = (companyType === 'priority' || isHalfReduction) ? (2/3) : (1/2);
            dailySubsidy = Math.min(dailyWage * rate, LIMIT);
        } else {
            // 무급: 1일 정액 기준 (승인된 계획에 따름, 여기선 한도액 기준)
            dailySubsidy = LIMIT;
        }

        const total = dailySubsidy * empCount * days;

        // 3. 결과 출력
        resArea.className = 'pass';
        resStatus.innerText = "✅ 지원 대상 예상";
        resMsg.innerText = `매출 요건(${revenueDrop}%)을 충족합니다. 1일 1인당 지원금: ${Math.round(dailySubsidy).toLocaleString()}원`;
        totalValue.innerText = Math.round(total).toLocaleString() + "원";
    }
</script>
</body>
</html>
