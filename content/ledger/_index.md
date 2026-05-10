---
title: "💰 账本"
date: 2026-05-10
---

## 复式记账

> 有借必有贷，借贷必相等 📚

<div id="ledger-app"></div>

<style>
.ledger-container { max-width: 100%; }
.ledger-container .form-row { display: flex; gap: 8px; flex-wrap: wrap; align-items: end; margin-bottom: 12px; }
.ledger-container .form-group { display: flex; flex-direction: column; gap: 4px; }
.ledger-container .form-group label { font-size: 0.8em; color: var(--secondary); font-weight: 600; }
.ledger-container .form-group input, .ledger-container .form-group select { padding: 6px 10px; border: 1px solid var(--border); border-radius: 6px; background: var(--entry); color: var(--primary); font-size: 0.9em; }
.ledger-container button { padding: 6px 16px; border: none; border-radius: 6px; background: var(--primary); color: var(--theme); cursor: pointer; font-weight: 600; font-size: 0.9em; }
.ledger-container button:hover { opacity: 0.8; }
.ledger-container .btn-danger { background: #e74c3c; }
.ledger-container .btn-sm { padding: 4px 10px; font-size: 0.8em; }
.ledger-container .summary { display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); gap: 10px; margin: 16px 0; }
.ledger-container .summary-card { border: 1px solid var(--border); border-radius: 8px; padding: 12px; text-align: center; background: var(--entry); }
.ledger-container .summary-card .label { font-size: 0.8em; color: var(--secondary); }
.ledger-container .summary-card .value { font-size: 1.4em; font-weight: bold; margin-top: 4px; }
.ledger-container .summary-card .value.positive { color: #2ecc71; }
.ledger-container .summary-card .value.negative { color: #e74c3c; }
.ledger-table { width: 100%; border-collapse: collapse; margin-top: 10px; }
.ledger-table th { padding: 8px 10px; border: 1px solid var(--border); text-align: left; font-size: 0.85em; background: var(--code-bg); }
.ledger-table td { padding: 8px 10px; border: 1px solid var(--border); font-size: 0.85em; }
.ledger-table .amount { text-align: right; font-family: monospace; }
.ledger-table .amount.positive { color: #2ecc71; }
.ledger-table .amount.negative { color: #e74c3c; }
.ledger-table tr:hover { background: var(--tertiary); }
.ledger-table .del-btn { cursor: pointer; color: #e74c3c; font-size: 1.1em; }
.empty-msg { text-align: center; padding: 40px; color: var(--secondary); }
</style>

<script>
(function() {
  const STORAGE_KEY = 'ledger_entries';
  let entries = [];

  function load() {
    try { entries = JSON.parse(localStorage.getItem(STORAGE_KEY)) || []; }
    catch(e) { entries = []; }
  }
  function save() { localStorage.setItem(STORAGE_KEY, JSON.stringify(entries)); }

  function today() {
    const d = new Date();
    return d.getFullYear() + '-' + String(d.getMonth()+1).padStart(2,'0') + '-' + String(d.getDate()).padStart(2,'0');
  }

  function render() {
    const app = document.getElementById('ledger-app');
    if (!app) return;
    load();
    
    let totalDebit = 0, totalCredit = 0;
    const accountBal = {};
    const accounts = ['现金','银行存款','支付宝','微信','应收款','应付账款','收入','支出','投资','其他'];
    accounts.forEach(a => accountBal[a] = 0);

    entries.forEach(e => {
      accountBal[e.debit] = (accountBal[e.debit] || 0) + parseFloat(e.amount);
      accountBal[e.credit] = (accountBal[e.credit] || 0) - parseFloat(e.amount);
      if (e.type === '收入') totalCredit += parseFloat(e.amount);
      else totalDebit += parseFloat(e.amount);
    });

    const balance = totalCredit - totalDebit;
    const acctList = Object.entries(accountBal).filter(([,v]) => v !== 0).sort((a,b) => Math.abs(b[1]) - Math.abs(a[1]));

    app.innerHTML = `
      <div class="ledger-container">
        <div class="form-row">
          <div class="form-group">
            <label>日期</label>
            <input type="date" id="l-date" value="${today()}">
          </div>
          <div class="form-group">
            <label>摘要</label>
            <input type="text" id="l-desc" placeholder="买什么/收入来源" style="width:160px;">
          </div>
          <div class="form-group">
            <label>类型</label>
            <select id="l-type"><option value="支出">支出</option><option value="收入">收入</option></select>
          </div>
          <div class="form-group">
            <label>借方科目</label>
            <select id="l-debit">
              ${accounts.map(a => '<option value="' + a + '">' + a + '</option>').join('')}
            </select>
          </div>
          <div class="form-group">
            <label>贷方科目</label>
            <select id="l-credit">
              ${accounts.map(a => '<option value="' + a + '">' + a + '</option>').join('')}
            </select>
          </div>
          <div class="form-group">
            <label>金额</label>
            <input type="number" id="l-amount" placeholder="0.00" step="0.01" min="0" style="width:100px;">
          </div>
          <div class="form-group" style="padding-bottom:2px;">
            <label>&nbsp;</label>
            <button onclick="addEntry()">➕ 记账</button>
          </div>
        </div>

        <div class="summary">
          <div class="summary-card">
            <div class="label">总收入</div>
            <div class="value positive">¥${totalCredit.toFixed(2)}</div>
          </div>
          <div class="summary-card">
            <div class="label">总支出</div>
            <div class="value negative">¥${totalDebit.toFixed(2)}</div>
          </div>
          <div class="summary-card">
            <div class="label">结余</div>
            <div class="value ${balance >= 0 ? 'positive' : 'negative'}">¥${balance.toFixed(2)}</div>
          </div>
          <div class="summary-card">
            <div class="label">总笔数</div>
            <div class="value">${entries.length}</div>
          </div>
        </div>

        ${acctList.length > 0 ? `
        <div style="margin:12px 0;">
          <div style="font-size:0.85em;font-weight:600;margin-bottom:6px;">📊 科目余额</div>
          <div style="display:flex;flex-wrap:wrap;gap:6px;">
            ${acctList.map(([k,v]) => '<span style="font-size:0.8em;padding:3px 10px;border:1px solid var(--border);border-radius:12px;background:var(--entry);">' + k + ': <b>' + (v >= 0 ? '' : '-') + '¥' + Math.abs(v).toFixed(2) + '</b></span>').join('')}
          </div>
        </div>` : ''}

        ${entries.length > 0 ? `
        <table class="ledger-table">
          <tr><th>日期</th><th>摘要</th><th>类型</th><th>借方</th><th>贷方</th><th style="text-align:right;">金额</th><th></th></tr>
          ${entries.slice().reverse().map((e,i) => {
            const idx = entries.length - 1 - i;
            return '<tr><td>' + e.date + '</td><td>' + e.desc + '</td><td><span style="font-size:0.8em;padding:1px 6px;border-radius:4px;background:' + (e.type === '收入' ? '#2ecc71' : '#e74c3c') + ';color:#fff;">' + e.type + '</span></td><td>' + e.debit + '</td><td>' + e.credit + '</td><td class="amount ' + (e.type === '收入' ? 'positive' : 'negative') + '">' + (e.type === '收入' ? '' : '-') + '¥' + parseFloat(e.amount).toFixed(2) + '</td><td><span class="del-btn" onclick="delEntry(' + idx + ')">✕</span></td></tr>';
          }).join('')}
        </table>` : '<div class="empty-msg">📒 暂无记账记录，开始记一笔吧～</div>'}
        
        ${entries.length > 0 ? '<div style="text-align:right;margin-top:8px;"><button class="btn-danger btn-sm" onclick="clearAll()">🗑️ 清空全部</button></div>' : ''}
      </div>
    `;
  }

  window.addEntry = function() {
    const date = document.getElementById('l-date').value;
    const desc = document.getElementById('l-desc').value.trim();
    const type = document.getElementById('l-type').value;
    const debit = document.getElementById('l-debit').value;
    const credit = document.getElementById('l-credit').value;
    const amount = parseFloat(document.getElementById('l-amount').value);
    
    if (!desc) { alert('请输入摘要'); return; }
    if (!amount || amount <= 0) { alert('请输入有效金额'); return; }
    if (debit === credit) { alert('借方和贷方不能相同！有借必有贷，借贷必相等 😅'); return; }

    entries.push({ date, desc, type, debit, credit, amount });
    save();
    render();
    document.getElementById('l-desc').value = '';
    document.getElementById('l-amount').value = '';
  };

  window.delEntry = function(idx) {
    if (confirm('确定删除这条记录？')) {
      entries.splice(idx, 1);
      save();
      render();
    }
  };

  window.clearAll = function() {
    if (confirm('确定清空全部记账记录？')) {
      entries = [];
      save();
      render();
    }
  };

  // 页面加载后渲染
  document.addEventListener('DOMContentLoaded', render);
  // 如果已经加载了，直接渲染
  if (document.getElementById('ledger-app')) render();
})();
</script>
