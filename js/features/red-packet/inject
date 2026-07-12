// ============================================================
// inject.js - 自动注入红包按钮（无需修改 listeners.js）
// ============================================================
(function() {
    'use strict';

    function injectButton() {
        if (document.getElementById('redpacket-btn')) return;

        var container = document.querySelector('.input-buttons');
        if (!container) return;

        var batchBtn = document.getElementById('batch-btn');
        if (!batchBtn) return;

        var rpBtn = document.createElement('button');
        rpBtn.id = 'redpacket-btn';
        rpBtn.className = 'input-btn collapse-hideable';
        rpBtn.title = '发红包';
        rpBtn.innerHTML = '<i class="fas fa-redo-alt" style="color: #e63e2e;"></i>';
        rpBtn.style.cssText = [
            'background: linear-gradient(135deg, #fce4c8, #f8d5b0);',
            'border: none; border-radius: 50%;',
            'width: 46px; height: 46px;',
            'cursor: pointer; display: flex; align-items: center; justify-content: center;',
            'flex-shrink: 0; box-shadow: 0 2px 8px rgba(230,62,46,0.25);',
            'transition: transform 0.2s, box-shadow 0.2s;'
        ].join(' ');
        rpBtn.onmouseover = function() {
            this.style.transform = 'scale(1.08)';
            this.style.boxShadow = '0 4px 16px rgba(230,62,46,0.4)';
        };
        rpBtn.onmouseout = function() {
            this.style.transform = 'scale(1)';
            this.style.boxShadow = '0 2px 8px rgba(230,62,46,0.25)';
        };
        rpBtn.onclick = function(e) {
            e.stopPropagation();
            if (window.Redpacket && typeof window.Redpacket.send === 'function') {
                window.Redpacket.send();
            } else {
                if (typeof showNotification === 'function') {
                    showNotification('红包模块未加载，请刷新页面', 'error');
                }
            }
        };

        // 插入到 batchBtn 后面
        batchBtn.parentNode.insertBefore(rpBtn, batchBtn.nextSibling);

        // 也添加到折叠面板（如果有）
        var extraPanel = document.getElementById('collapsed-extras-panel');
        if (extraPanel) {
            var extraGrid = extraPanel.querySelector('.collapsed-extras-inner > div');
            if (extraGrid) {
                var extraRpBtn = document.createElement('button');
                extraRpBtn.className = 'collapsed-extra-btn';
                extraRpBtn.title = '发红包';
                extraRpBtn.innerHTML = '<i class="fas fa-redo-alt" style="color:#e63e2e;"></i>';
                extraRpBtn.onclick = function(e) {
                    e.stopPropagation();
                    document.getElementById('collapsed-extras-panel').style.display = 'none';
                    document.getElementById('collapse-expand-btn').classList.remove('open');
                    if (window.Redpacket && typeof window.Redpacket.send === 'function') {
                        window.Redpacket.send();
                    }
                };
                extraGrid.appendChild(extraRpBtn);
            }
        }
    }

    // 页面加载完成后注入
    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', injectButton);
    } else {
        injectButton();
    }

    // 也监听动态变化（以防万一）
    var observer = new MutationObserver(function() {
        if (!document.getElementById('redpacket-btn')) {
            injectButton();
        }
    });
    observer.observe(document.body, { childList: true, subtree: true });
})();
