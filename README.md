<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>文本检查工具</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', system-ui, sans-serif;
        }
        
        body {
            background: #f0f2f5;
            min-height: 100vh;
            padding: 20px;
            color: #1a1a1a;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        
        .container {
            width: 100%;
            max-width: 1000px;
            background: white;
            border-radius: 12px;
            box-shadow: 0 4px 20px rgba(0, 0, 0, 0.12);
            overflow: hidden;
        }
        
        header {
            background: #2c3e50;
            color: white;
            padding: 20px;
            text-align: center;
        }
        
        h1 {
            font-size: 1.8rem;
            font-weight: 600;
            margin: 0;
        }
        
        .main-content {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
            padding: 20px;
        }
        
        @media (max-width: 768px) {
            .main-content {
                grid-template-columns: 1fr;
            }
        }
        
        .section {
            margin-bottom: 20px;
        }
        
        h2 {
            font-size: 1.3rem;
            margin-bottom: 12px;
            color: #2c3e50;
            padding-bottom: 8px;
            border-bottom: 2px solid #3498db;
        }
        
        textarea {
            width: 100%;
            height: 250px;
            padding: 15px;
            border: 1px solid #ddd;
            border-radius: 8px;
            font-size: 1rem;
            line-height: 1.5;
            resize: vertical;
            background: #fafafa;
        }
        
        textarea:focus {
            outline: none;
            border-color: #3498db;
            box-shadow: 0 0 0 2px rgba(52, 152, 219, 0.2);
        }
        
        .actions {
            display: flex;
            gap: 10px;
            margin: 15px 0;
        }
        
        button {
            padding: 10px 16px;
            border: none;
            border-radius: 6px;
            font-size: 0.95rem;
            font-weight: 500;
            cursor: pointer;
            transition: all 0.2s;
        }
        
        .btn-check {
            background: #3498db;
            color: white;
            flex: 1;
        }
        
        .btn-check:hover {
            background: #2980b9;
        }
        
        .btn-clear {
            background: #e74c3c;
            color: white;
        }
        
        .btn-clear:hover {
            background: #c0392b;
        }
        
        .result-container {
            background: #fafafa;
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
            min-height: 250px;
            line-height: 1.5;
            font-size: 1rem;
            overflow-wrap: break-word;
        }
        
        .mixed-highlight {
            background-color: #ffeaa7;
            padding: 0 2px;
            border-radius: 3px;
            border-bottom: 2px solid #fdcb6e;
        }
        
        .asterisk-error {
            background-color: #ffcccc;
            padding: 0 2px;
            border-radius: 3px;
            border-bottom: 2px solid #ff6b6b;
        }
        
        .stats {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            margin-top: 15px;
        }
        
        .stat-box {
            background: #f8f9fa;
            border-radius: 8px;
            padding: 12px;
            text-align: center;
            border-left: 4px solid #3498db;
        }
        
        .stat-title {
            font-size: 0.9rem;
            color: #7f8c8d;
            margin-bottom: 5px;
        }
        
        .stat-value {
            font-size: 1.5rem;
            font-weight: 700;
            color: #2c3e50;
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>文本检查工具</h1>
        </header>
        
        <div class="main-content">
            <div class="section">
                <h2>输入文本</h2>
                <textarea id="inputText">这个APP的功能非常powerful，可以帮你manage你的daily tasks。使用它之后，我的productivity有了明显improvement。

**注意：这个文本中包含一些星号错误：
- 不成对星号：这里有一个*单独的星号
- 格式错误：**错误示例1*，*错误示例2**
- 正确的加粗：**这是正确格式**</textarea>
                
                <div class="actions">
                    <button id="checkBtn" class="btn-check">检查文本</button>
                    <button id="clearBtn" class="btn-clear">清空</button>
                </div>
            </div>
            
            <div class="section">
                <h2>检查结果</h2>
                <div class="result-container" id="resultContainer">
                    检查结果将显示在这里
                </div>
                
                <div class="stats">
                    <div class="stat-box">
                        <div class="stat-title">中英混杂数量</div>
                        <div class="stat-value" id="mixedCount">0</div>
                    </div>
                    <div class="stat-box">
                        <div class="stat-title">星号错误数量</div>
                        <div class="stat-value" id="asteriskErrorCount">0</div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            const inputText = document.getElementById('inputText');
            const checkBtn = document.getElementById('checkBtn');
            const clearBtn = document.getElementById('clearBtn');
            const resultContainer = document.getElementById('resultContainer');
            const mixedCount = document.getElementById('mixedCount');
            const asteriskErrorCount = document.getElementById('asteriskErrorCount');
            
            // 初始检查
            checkText();
            
            // 检查按钮事件
            checkBtn.addEventListener('click', checkText);
            
            // 清空按钮事件
            clearBtn.addEventListener('click', function() {
                inputText.value = '';
                resultContainer.textContent = '检查结果将显示在这里';
                mixedCount.textContent = '0';
                asteriskErrorCount.textContent = '0';
            });
            
            // 检查文本函数
            function checkText() {
                const text = inputText.value;
                if (!text.trim()) {
                    resultContainer.textContent = '请输入需要检查的文本内容';
                    mixedCount.textContent = '0';
                    asteriskErrorCount.textContent = '0';
                    return;
                }
                
                // 检测中英混合模式
                const mixedPattern = /[\u4e00-\u9fa5][a-zA-Z]+|[a-zA-Z]+[\u4e00-\u9fa5]/g;
                let match;
                const mixedPositions = [];
                
                while ((match = mixedPattern.exec(text)) !== null) {
                    mixedPositions.push({
                        start: match.index,
                        end: match.index + match[0].length,
                        text: match[0],
                        type: 'mixed'
                    });
                }
                
                // 检测星号渲染错误（所有不成对或格式错误的星号）
                const asteriskErrors = [];
                
                // 检测所有星号使用情况
                const asteriskPattern = /(\*+)([^*]*?)(\*+)?/g;
                while ((match = asteriskPattern.exec(text)) !== null) {
                    const fullMatch = match[0];
                    const openingAsterisks = match[1];
                    const closingAsterisks = match[3] || '';
                    
                    // 情况1：只有开头星号没有结尾星号
                    if (!closingAsterisks) {
                        asteriskErrors.push({
                            start: match.index,
                            end: match.index + fullMatch.length,
                            text: fullMatch,
                            type: 'asterisk'
                        });
                    } 
                    // 情况2：开头和结尾星号数量不匹配
                    else if (openingAsterisks.length !== closingAsterisks.length) {
                        asteriskErrors.push({
                            start: match.index,
                            end: match.index + fullMatch.length,
                            text: fullMatch,
                            type: 'asterisk'
                        });
                    }
                }
                
                // 更新统计信息
                mixedCount.textContent = mixedPositions.length;
                asteriskErrorCount.textContent = asteriskErrors.length;
                
                // 合并所有问题并按位置排序
                const allIssues = [...mixedPositions, ...asteriskErrors].sort((a, b) => a.start - b.start);
                
                // 显示检查结果
                if (allIssues.length === 0) {
                    resultContainer.textContent = '未发现任何问题，文本规范！';
                } else {
                    let highlightedText = '';
                    let lastIndex = 0;
                    
                    allIssues.forEach(issue => {
                        // 添加问题点之前的内容
                        highlightedText += text.substring(lastIndex, issue.start);
                        
                        // 添加高亮的问题内容
                        const highlightClass = issue.type === 'mixed' ? 'mixed-highlight' : 'asterisk-error';
                        highlightedText += `<span class="${highlightClass}">${text.substring(issue.start, issue.end)}</span>`;
                        
                        lastIndex = issue.end;
                    });
                    
                    // 添加剩余文本
                    highlightedText += text.substring(lastIndex);
                    
                    resultContainer.innerHTML = highlightedText;
                }
            }
        });
    </script>
</body>
</html>
