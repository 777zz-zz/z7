// ============================================================
// redpacket.js - 红包功能模块（独立文件夹版）
// 依赖：全局 messages, settings, addMessage, renderMessages,
//       throttledSaveData, showNotification, playSound
// ============================================================
(function() {
    'use strict';

    // ---------- 祝福文案 ----------
    var BLESSINGS = {
        special: {
            '520.13': '我爱你一生一世 ❤️',
            '520.1314': '我爱你一生一世 ❤️',
            '13.14': '一生一世 💕',
            '5.20': '我爱你 💖',
            '52.00': '我爱你 💗',
            '20.13': '爱你一生 🌹',
            '88.88': '发发发发 🧧',
            '99.99': '长长久久 💞',
            '66.66': '顺顺利利 🎋',
            '11.11': '一心一意 💝',
            '33.44': '生生世世 🌹'
        },
        normal: [
            '祝你今天开心 ✨', '一切都会好的 🌸', '愿好运常伴 🍀',
            '天天都有好心情 ☀️', '你是最棒的 💪', '幸福满满 🥰',
            '平安喜乐 🌙', '万事胜意 🌟', '前程似锦 🎈',
            '岁岁平安 🎋', '心想事成 💫', '阳光万里 ☀️',
            '未来可期 🌈', '今天也很想你 💭', '要一直开心哦 🥺',
            '爱你呀 💕', '遇见你真好 🌹', '晚安好梦 🌙'
        ]
    };

    // ---------- 特殊金额权重 ----------
    var SPECIAL_AMOUNTS = [
        { amount: 520.13, weight: 10 },
        { amount: 520.1314, weight: 8 },
        { amount: 13.14, weight: 8 },
        { amount: 5.20, weight: 6 },
        { amount: 52.00, weight: 5 },
        { amount: 20.13, weight: 5 },
        { amount: 88.88, weight: 4 },
        { amount: 99.99, weight: 4 },
        { amount: 66.66, weight: 4 },
        { amount: 11.11, weight: 4 },
        { amount: 33.44, weight: 3 }
    ];
    var SPECIAL_TOTAL_WEIGHT = SPECIAL_AMOUNTS.reduce(function(s, i) { return s + i.weight; }, 0);

    // ---------- 生成红包 ----------
    function generateRedpacket() {
        var isSpecial = Math.random() < 0.45;
        var amount, blessing;
        if (isSpecial) {
            var rand = Math.random() * SPECIAL_TOTAL_WEIGHT;
            for (var i = 0; i < SPECIAL_AMOUNTS.length; i++) {
                rand -= SPECIAL_AMOUNTS[i].weight;
                if (rand <= 0) { amount = SPECIAL_AMOUNTS[i].amount; break; }
            }
            if (!amount) amount = SPECIAL_AMOUNTS[0].amount;
            var key = String(amount);
            blessing = BLESSINGS.special[key] ||
                       BLESSINGS.normal[Math.floor(Math.random() * BLESSINGS.normal.length)];
        } else {
            amount = parseFloat((Math.random() * 99.99 + 0.01).toFixed(2));
            blessing = BLESSINGS.normal[Math.floor(Math.random() * BLESSINGS.normal.length)];
        }
        return { amount: amount, blessing: blessing };
    }

    // ---------- 发送红包 ----------
    function sendRedpacket() {
        var rp = generateRedpacket();
        var msg = {
            id: Date.now() + Math.random(),
            sender: 'user',
            type: 'redpacket',
            timestamp: new Date(),
            redpacket: {
                amount: rp.amount,
                blessing: rp.blessing,
                status: 'pending',
                sentTime: Date.now(),
                claimTime: null,
                returnTime: null,
                expireTime: Date.now() + 24 * 60 * 60 * 1000,
                claimedBy: null,
                returnedBy: null
            }
        };
        if (typeof addMessage === 'function') {
            addMessage(msg);
        } else {
            // 降级
            messages.push(msg);
            if (typeof renderMessages === 'function') renderMessages();
            if (typeof throttledSaveData === 'function') throttledSaveData();
        }
        if (typeof showNotification === 'function') {
            showNotification('🧧 红包已发出！', 'success', 2000);
        }
        if (typeof playSound === 'function') playSound('send');
        startExpiryChecker();
    }

    // ---------- 查找红包消息 ----------
    function findRedpacketMessage(msgId) {
        if (typeof messages === 'undefined' || !messages) return null;
        for (var i = 0; i < messages.length; i++) {
            if (messages[i].id === msgId && messages[i].type === 'redpacket') {
                return messages[i];
            }
        }
        return null;
    }

    // ---------- 领取 ----------
    function claimRedpacket(msgId) {
        var msg = findRedpacketMessage(msgId);
        if (!msg) return;
        if (msg.redpacket.status !== 'pending') {
            showNotification('这个红包已经处理过了', 'info');
            return;
        }
        if (msg.sender === 'user') {
            showNotification('这是你发的红包，不能自己领哦', 'warning');
            return;
        }
        // 20% 延迟领取
        if (Math.random() < 0.2) {
            var delay = 30 * 1000 + Math.random() * 30 * 60 * 1000;
            var delayMin = Math.round(delay / 60000);
            showNotification('💭 对方正在考虑要不要收下… 预计 ' + (delayMin > 0 ? delayMin + ' 分钟后' : '一会') + ' 领取', 'info', 3000);
            setTimeout(function() { doClaimRedpacket(msgId); }, delay);
            return;
        }
        doClaimRedpacket(msgId);
    }

    function doClaimRedpacket(msgId) {
        var msg = findRedpacketMessage(msgId);
        if (!msg || msg.redpacket.status !== 'pending') return;
        msg.redpacket.status = 'claimed';
        msg.redpacket.claimTime = Date.now();
        msg.redpacket.claimedBy = 'partner';

        var partnerName = (typeof settings !== 'undefined' && settings.partnerName) || '对方';
        var sysMsg = {
            id: Date.now() + Math.random(),
            sender: 'system',
            type: 'normal',
            text: '🧧 ' + partnerName + ' 领取了你的红包 ¥' + msg.redpacket.amount.toFixed(2) + '  ' + msg.redpacket.blessing,
            timestamp: new Date()
        };
        messages.push(sysMsg);
        if (typeof throttledSaveData === 'function') throttledSaveData();
        if (typeof renderMessages === 'function') renderMessages();
        if (typeof showNotification === 'function') {
            showNotification('🧧 红包已领取 ¥' + msg.redpacket.amount.toFixed(2), 'success', 3000);
        }
        if (typeof playSound === 'function') playSound('favorite');
    }

    // ---------- 退回 ----------
    function returnRedpacketBySender(msgId) {
        var msg = findRedpacketMessage(msgId);
        if (!msg) return;
        if (msg.redpacket.status !== 'pending') { showNotification('红包已处理，无法退回', 'warning'); return; }
        if (msg.sender !== 'user') { showNotification('只有发送者可以退回', 'warning'); return; }
        if (Date.now() - msg.redpacket.sentTime > 24 * 60 * 60 * 1000) {
            showNotification('已超过24小时，红包已自动退回', 'info');
            return;
        }
        doReturnRedpacket(msgId, 'user', '你已退回红包 ¥' + msg.redpacket.amount.toFixed(2));
    }

    function returnRedpacketByReceiver(msgId) {
        var msg = findRedpacketMessage(msgId);
        if (!msg) return;
        if (msg.redpacket.status !== 'pending') { showNotification('红包已处理，无法退回', 'warning'); return; }
        if (msg.sender === 'user') { showNotification('这是你发的红包，不能退回', 'warning'); return; }
        doReturnRedpacket(msgId, 'partner', '你已退回红包 ¥' + msg.redpacket.amount.toFixed(2));
    }

    function doReturnRedpacket(msgId, who, toastMsg) {
        var msg = findRedpacketMessage(msgId);
        if (!msg || msg.redpacket.status !== 'pending') return;
        var statusKey = (who === 'user') ? 'returned_by_sender' : 'returned_by_receiver';
        msg.redpacket.status = statusKey;
        msg.redpacket.returnTime = Date.now();
        msg.redpacket.returnedBy = who;

        var sysMsg = {
            id: Date.now() + Math.random(),
            sender: 'system',
            type: 'normal',
            text: '🧧 ' + toastMsg + '  ' + msg.redpacket.blessing,
            timestamp: new Date()
        };
        messages.push(sysMsg);
        if (typeof throttledSaveData === 'function') throttledSaveData();
        if (typeof renderMessages === 'function') renderMessages();
        if (typeof showNotification === 'function') showNotification('🧧 ' + toastMsg, 'info', 2500);
        if (typeof playSound === 'function') playSound('error');
    }

    // ---------- 过期检查 ----------
    var expiryTimer = null;
    function startExpiryChecker() {
        if (expiryTimer) return;
        expiryTimer = setInterval(function() {
            var now = Date.now();
            var changed = false;
            if (typeof messages === 'undefined' || !messages) return;
            for (var i = 0; i < messages.length; i++) {
                var msg = messages[i];
                if (msg.type !== 'redpacket') continue;
                if (msg.redpacket.status !== 'pending') continue;
                if (now >= msg.redpacket.expireTime) {
                    msg.redpacket.status = 'expired';
                    msg.redpacket.returnTime = now;
                    msg.redpacket.returnedBy = 'system';
                    var partnerName = (typeof settings !== 'undefined' && settings.partnerName) || '对方';
                    var sysMsg = {
                        id: Date.now() + Math.random() + i,
                        sender: 'system',
                        type: 'normal',
                        text: '🧧 红包已过期，自动退回 ¥' + msg.redpacket.amount.toFixed(2) + '  ' + msg.redpacket.blessing,
                        timestamp: new Date(now)
                    };
                    messages.push(sysMsg);
                    changed = true;
                }
            }
            if (changed) {
                if (typeof throttledSaveData === 'function') throttledSaveData();
                if (typeof renderMessages === 'function') renderMessages();
            }
        }, 60 * 1000);
    }

    // ---------- 获取状态文字 ----------
    function getStatusText(status, rp) {
        switch (status) {
            case 'pending': return '待领取';
            case 'claimed': return '已领取 ¥' + rp.amount.toFixed(2);
            case 'returned_by_sender': return '已退回 ¥' + rp.amount.toFixed(2);
            case 'returned_by_receiver': return '已退回 ¥' + rp.amount.toFixed(2);
            case 'expired': return '已过期退回 ¥' + rp.amount.toFixed(2);
            default: return '';
        }
    }

    function getStatusColor(status) {
        switch (status) {
            case 'pending': return 'var(--accent-color)';
            case 'claimed': return '#4CAF50';
            case 'returned_by_sender':
            case 'returned_by_receiver':
            case 'expired': return 'var(--text-secondary)';
            default: return 'var(--text-secondary)';
        }
    }

    // ---------- 渲染红包卡片（适配网站美化） ----------
    function renderCard(msg, bubbleClass) {
        var rp = msg.redpacket;
        var status = rp.status;
        var isSender = msg.sender === 'user';
        var isPending = status === 'pending';
        var isClaimed = status === 'claimed';
        var isReturned = status === 'returned_by_sender' || status === 'returned_by_receiver' || status === 'expired';

        var card = document.createElement('div');
        card.className = 'message redpacket-card ' + (bubbleClass || '');
        card.style.cssText = [
            'padding: 12px 16px !important;',
            'background: var(--secondary-bg) !important;',
            'border: 1px solid var(--border-color) !important;',
            'border-radius: var(--radius) !important;',
            'box-shadow: var(--shadow) !important;',
            'min-width: 160px;',
            'max-width: 220px;',
            'cursor: pointer;',
            'transition: transform 0.2s ease, box-shadow 0.2s ease;',
            'position: relative;',
            'overflow: hidden;'
        ].join(' ');
        card.onmouseover = function() { this.style.transform = 'translateY(-2px)'; this.style.boxShadow = 'var(--shadow)'; };
        card.onmouseout = function() { this.style.transform = ''; this.style.boxShadow = 'var(--shadow)'; };

        // ---- 顶部：发送者 + 金额 ----
        var topRow = document.createElement('div');
        topRow.style.cssText = 'display: flex; justify-content: space-between; align-items: baseline; margin-bottom: 4px;';
        var senderName = (msg.sender === 'user')
            ? ((typeof settings !== 'undefined' && settings.myName) || '我')
            : ((typeof settings !== 'undefined' && settings.partnerName) || '对方');
        var senderSpan = document.createElement('span');
        senderSpan.style.cssText = 'font-size: 12px; color: var(--text-secondary);';
        senderSpan.textContent = '来自 ' + senderName + ' 的 ';
        topRow.appendChild(senderSpan);

        var amountSpan = document.createElement('span');
        amountSpan.style.cssText = 'font-size: 18px; font-weight: 700; color: var(--accent-color);';
        amountSpan.textContent = '¥' + rp.amount.toFixed(2);
        topRow.appendChild(amountSpan);
        card.appendChild(topRow);

        // ---- 中间：状态标签 ----
        var statusText = getStatusText(status, rp);
        var statusColor = getStatusColor(status);
        var statusEl = document.createElement('div');
        statusEl.style.cssText = [
            'font-size: 13px;',
            'font-weight: 600;',
            'color: ' + statusColor + ';',
            'margin: 2px 0 4px;',
            'letter-spacing: 0.3px;'
        ].join(' ');
        statusEl.textContent = statusText;
        card.appendChild(statusEl);

        // ---- 祝福文案 ----
        var blessEl = document.createElement('div');
        blessEl.style.cssText = [
            'font-size: 13px;',
            'color: var(--text-secondary);',
            'font-style: italic;',
            'margin: 2px 0 6px;',
            'opacity: 0.85;'
        ].join(' ');
        blessEl.textContent = rp.blessing;
        card.appendChild(blessEl);

        // ---- 底部：时间 ----
        var timeEl = document.createElement('div');
        timeEl.style.cssText = 'font-size: 11px; color: var(--text-secondary); opacity: 0.6; text-align: right;';
        var ts = new Date(msg.timestamp);
        timeEl.textContent = ts.toLocaleTimeString('zh-CN', { hour: '2-digit', minute: '2-digit' });
        card.appendChild(timeEl);

        // ---- 如果是待领取且不是发送者，显示“点击领取”提示 ----
        if (isPending && !isSender) {
            var hint = document.createElement('div');
            hint.style.cssText = [
                'position: absolute;',
                'bottom: 4px;',
                'right: 8px;',
                'font-size: 10px;',
                'color: var(--accent-color);',
                'opacity: 0.7;',
                'font-weight: 500;'
            ].join(' ');
            hint.textContent = '点击领取';
            card.appendChild(hint);
        }

        // 存储红包ID
        card.dataset.rpId = msg.id;
        return card;
    }

    // ---------- 显示详情弹窗（含领取/退回按钮） ----------
    function showDetail(msgId) {
        var msg = findRedpacketMessage(msgId);
        if (!msg) { showNotification('红包不存在', 'error'); return; }
        var rp = msg.redpacket;
        var isSender = msg.sender === 'user';
        var isPending = rp.status === 'pending';
        var isClaimable = isPending && !isSender;
        var isReturnableBySender = isPending && isSender && (Date.now() - rp.sentTime < 24 * 60 * 60 * 1000);

        var overlay = document.createElement('div');
        overlay.style.cssText = [
            'position: fixed; inset: 0; z-index: 99999;',
            'background: rgba(0,0,0,0.6); backdrop-filter: blur(8px);',
            'display: flex; align-items: center; justify-content: center;',
            'animation: fadeIn 0.25s ease;'
        ].join(' ');

        var panel = document.createElement('div');
        panel.style.cssText = [
            'background: var(--secondary-bg);',
            'border-radius: var(--radius);',
            'padding: 28px 24px 22px;',
            'width: min(340px, 88vw);',
            'box-shadow: 0 24px 60px rgba(0,0,0,0.4);',
            'text-align: center;',
            'position: relative;',
            'border: 1px solid var(--border-color);'
        ].join(' ');

        // 关闭按钮
        var closeBtn = document.createElement('button');
        closeBtn.textContent = '×';
        closeBtn.style.cssText = 'position:absolute; top:10px; right:14px; background:none; border:none; font-size:22px; color:var(--text-secondary); cursor:pointer;';
        closeBtn.onclick = function() { overlay.remove(); };
        panel.appendChild(closeBtn);

        // 金额
        var amt = document.createElement('div');
        amt.style.cssText = 'font-size: 38px; font-weight: 700; color: var(--accent-color); margin: 4px 0;';
        amt.textContent = '¥' + rp.amount.toFixed(2);
        panel.appendChild(amt);

        // 状态
        var st = document.createElement('div');
        st.style.cssText = 'font-size: 14px; font-weight: 600; color: ' + getStatusColor(rp.status) + '; margin-bottom: 4px;';
        st.textContent = getStatusText(rp.status, rp);
        panel.appendChild(st);

        // 祝福
        var bl = document.createElement('div');
        bl.style.cssText = 'font-size: 15px; color: var(--text-secondary); font-style: italic; margin: 8px 0 16px;';
        bl.textContent = '「' + rp.blessing + '」';
        panel.appendChild(bl);

        // 时间信息
        var timeInfo = document.createElement('div');
        timeInfo.style.cssText = 'font-size: 11px; color: var(--text-secondary); opacity: 0.6; margin-bottom: 16px; line-height:1.6;';
        var lines = ['发送于 ' + new Date(rp.sentTime).toLocaleString('zh-CN', {month:'2-digit', day:'2-digit', hour:'2-digit', minute:'2-digit'})];
        if (rp.claimTime) lines.push('领取于 ' + new Date(rp.claimTime).toLocaleString('zh-CN', {month:'2-digit', day:'2-digit', hour:'2-digit', minute:'2-digit'}));
        if (rp.returnTime) lines.push('退回于 ' + new Date(rp.returnTime).toLocaleString('zh-CN', {month:'2-digit', day:'2-digit', hour:'2-digit', minute:'2-digit'}));
        timeInfo.textContent = lines.join('  ·  ');
        panel.appendChild(timeInfo);

        // 操作按钮
        var actions = document.createElement('div');
        actions.style.cssText = 'display: flex; gap: 10px; flex-wrap: wrap; justify-content: center; margin-top: 4px;';

        if (isClaimable) {
            var claimBtn = document.createElement('button');
            claimBtn.textContent = '🧧 领取';
            claimBtn.style.cssText = 'flex:1; padding:10px 20px; border:none; border-radius:12px; background:var(--accent-color); color:#fff; font-size:16px; font-weight:700; cursor:pointer; transition:transform 0.2s;';
            claimBtn.onmouseover = function() { this.style.transform = 'scale(1.04)'; };
            claimBtn.onmouseout = function() { this.style.transform = 'scale(1)'; };
            claimBtn.onclick = function() { overlay.remove(); claimRedpacket(msgId); };
            actions.appendChild(claimBtn);

            var rejectBtn = document.createElement('button');
            rejectBtn.textContent = '退回';
            rejectBtn.style.cssText = 'flex:0.6; padding:10px 16px; border:1px solid var(--border-color); border-radius:12px; background:var(--primary-bg); color:var(--text-secondary); font-size:14px; cursor:pointer;';
            rejectBtn.onmouseover = function() { this.style.background = 'var(--border-color)'; };
            rejectBtn.onmouseout = function() { this.style.background = 'var(--primary-bg)'; };
            rejectBtn.onclick = function() {
                if (confirm('确定退回这个红包吗？')) { overlay.remove(); returnRedpacketByReceiver(msgId); }
            };
            actions.appendChild(rejectBtn);
        }

        if (isReturnableBySender) {
            var returnBtn = document.createElement('button');
            returnBtn.textContent = '↩️ 退回红包';
            returnBtn.style.cssText = 'flex:1; padding:10px 20px; border:1px solid #e74c3c; border-radius:12px; background:rgba(231,76,60,0.08); color:#e74c3c; font-size:14px; font-weight:600; cursor:pointer;';
            returnBtn.onmouseover = function() { this.style.background = 'rgba(231,76,60,0.18)'; };
            returnBtn.onmouseout = function() { this.style.background = 'rgba(231,76,60,0.08)'; };
            returnBtn.onclick = function() {
                if (confirm('确定退回这个红包吗？对方将无法领取。')) { overlay.remove(); returnRedpacketBySender(msgId); }
            };
            actions.appendChild(returnBtn);
        }

        if (!isClaimable && !isReturnableBySender) {
            var closeOnly = document.createElement('button');
            closeOnly.textContent = '关闭';
            closeOnly.style.cssText = 'padding:10px 32px; border:1px solid var(--border-color); border-radius:12px; background:var(--primary-bg); color:var(--text-secondary); cursor:pointer;';
            closeOnly.onclick = function() { overlay.remove(); };
            actions.appendChild(closeOnly);
        }

        panel.appendChild(actions);
        overlay.appendChild(panel);
        document.body.appendChild(overlay);

        overlay.addEventListener('click', function(e) {
            if (e.target === overlay) overlay.remove();
        });
    }

    // ---------- 暴露全局接口 ----------
    window.Redpacket = {
        send: sendRedpacket,
        claim: claimRedpacket,
        returnBySender: returnRedpacketBySender,
        returnByReceiver: returnRedpacketByReceiver,
        showDetail: showDetail,
        renderCard: renderCard,
        findMessage: findRedpacketMessage,
        startChecker: startExpiryChecker
    };

    // 自动启动过期检查
    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', function() { setTimeout(startExpiryChecker, 3000); });
    } else {
        setTimeout(startExpiryChecker, 3000);
    }

})();
