import { Resend } from 'resend';

// 邮件API密钥
const resend = new Resend('填入API密钥');

// 读取上一次状态
const LAST_STATE_KEY = "last_state";

// 模拟真实请求头，避免被防火墙拦截
const headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.81 Safari/537.36',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8',
    'Accept-Language': 'en-US,en;q=0.9',
    'Accept-Encoding': 'gzip, deflate, br',
    'Connection': 'keep-alive',
    'Upgrade-Insecure-Requests': '1',
    'TE': 'Trailers'
};

// 正则映射，用于从页面内容中提取信息
const regexMap = {
    colleges: /<td\s+align="right">院校名称：<\/td>\s*<td\s+align="left">\s*<b[^>]*>([^<]*)<\/b>\s*<\/td>/,
    majors: /<td\s+align="right">专业名称：<\/td>\s*<td\s+align="left">\s*<b[^>]*>([^<]*)<\/b>\s*<\/td>/,
    batches: /<td\s+align="right">批次：<\/td>\s*<td\s+align="left">\s*<b[^>]*>([^<]*)<\/b>\s*<\/td>/,
    status: /<td\s+align="right">考生状态：<\/td>\s*<td\s+align="left">\s*<b[^>]*>([^<]*)<\/b>\s*<\/td>/
};

// 主进程，处理定时触发的事件
async function handleScheduled(event) {
    const url = "填入自己的查询网址";

    try {
        // 发起 HTTP 请求获取网页内容
        const response = await fetch(url, { headers });
        if (!response.ok) throw new Error(`HTTP 错误! 状态码: ${response.status}`);

        // 获取并清理页面文本内容
        let text = (await response.text()).replace(/\s+/g, ' ');

        // 检查是否有拦截或防火墙相关内容
        if (/拦截|防火墙|安全|错误/.test(text)) {
            await sendEmail('拦截或防火墙警报', '检测到拦截或防火墙相关内容,请注意检查程序。');
            return;
        }

        // 提取页面中的信息
        const data = extractInformation(text);
        if (data.status) {
            // 获取上一次记录的状态
            const lastState = await KV.get(LAST_STATE_KEY);
            if (data.status !== lastState) {
                // 如果状态发生变化，则更新状态并发送邮件通知
                await KV.put(LAST_STATE_KEY, data.status);
                await sendEmail('录取进程更新通知', formatUpdateEmailContent(data));
            }
        }
    } catch (error) {
        console.error('主进程错误：', error); // 记录错误信息
    }
}

// 从页面文本中提取信息
function extractInformation(text) {
    const extractWithRegex = (regex, name) => {
        const match = regex.exec(text);

        const result = match ? match[1].trim() : '';
        console.log(`${name}:`, result); // 打印每个提取结果用于调试
        
        return result;
    };

    const data = Object.keys(regexMap).reduce((acc, key) => {
        acc[key] = extractWithRegex(regexMap[key], key);
        return acc;
    }, {});

    // 处理空值
    for (const key in data) {
        if (data[key] === '') {
            data[key] = '暂无信息';
        }
    }

    return data;
}

// 发送邮件通知
async function sendEmail(subject, content) {
    try {
        const { data, error } = await resend.emails.send({
            from: '录取进程通知 <发信邮箱>',
            to: ['收信邮箱'],
            subject,
            text: content,
            html: `<div style="line-height: 1.6;"><p>${content}</p></div>`,
        });

        if (error) throw new Error(`发送邮件失败：${error.message}`);
        return data;
    } catch (error) {
        console.error('邮件发送失败:', error); // 记录错误信息
        throw error;
    }
}

// 格式化更新邮件内容
function formatUpdateEmailContent(data) {
    return `
        <p>您好！<br>您的录取进程已更新：</p><br>
        <b>院校名称：</b>${data.colleges}<br>
        <b>专业名称：</b>${data.majors}<br>
        <b>录取批次：</b>${data.batches}<br>
        <b>考生状态：</b>${data.status}<br>
        <br>请及时<a href="填入自己的查询网址">登录系统</a>查看详情。
    `;
}

// 事件监听器，用于处理定时触发事件
addEventListener('scheduled', (event) => {
    event.waitUntil(handleScheduled(event));
});
