---
title: 💻 Cocos H5 Web/Mobile 逆向：UUID 解码与一键下载小游戏
published: 2025-12-04
updated: 2025-12-04
description: 'Cocos H5 Web/Mobile 逆向：UUID 解码与一键下载小游戏'
image: '/images/cocos_h5_reverse.png'
tags: [cocos, 逆向]
category: '逆向'
draft: false 
---
## 💻 Cocos H5 Web/Mobile 逆向：UUID 解码与一键下载小游戏

> **声明：** 本文档仅用于技术研究与交流，请勿用于非法用途。逆向工程应遵守相关法律法规及软件许可协议。

### 🚀 一、项目目标与技术栈

本项目旨在针对 Cocos Creator H5 小游戏，实现资源文件的批量解析和下载。

* **配置来源：** N 个 `config.json` 配置文件。
* **实现语言：** PHP。

### 🔍 二、UUID 解码与资源数据整合

由于目标语言是 PHP，所有的资源处理逻辑都将使用 PHP 实现。

#### 1. PHP UUID 解码函数

实现 Cocos 特定的 UUID 解码逻辑，将短 UUID 还原为完整的 32 位 UUID。

**【代码预留区域 - PHP UUID 短码解码函数】**

```php
<?php
// 配置选项
define('OUTPUT_FILE', '1.txt');  // 下载地址输出文件
define('MAX_WORKERS', 5);                    // 最大工作进程数

function getSubdirectories($path, $absolutePath = false) {
    if (!is_dir($path) || !is_readable($path)) {
        throw new InvalidArgumentException("无效目录或目录不可读: $path");
    }

    $path = rtrim($path, '/\\') . DIRECTORY_SEPARATOR;
    $directories = glob($path . '*', GLOB_ONLYDIR | GLOB_MARK);
    
    if (!$absolutePath) {
        $directories = array_map(function($dir) use ($path) {
            return str_replace($path, '', $dir);
        }, $directories);
    }

    return array_map(function($dir) {
        return rtrim($dir, '/\\');
    }, $directories);
}

function findAllConfigFilesRecursively($directory) {
    $configFiles = [];
    
    $directory = rtrim($directory, '/') . '/';
    $handle = opendir($directory);
    
    while (($file = readdir($handle)) !== false) {
        if ($file === '.' || $file === '..') continue;
        
        $fullPath = $directory . $file;
        
        if (is_file($fullPath) && preg_match('/^config\.[a-zA-Z0-9]+\.json$/', $file)) {
            $configFiles[] = [
                'file' => $file,
                'path' => $fullPath,
                'relative_path' => $file,
                'directory' => $directory
            ];
        }
        elseif (is_dir($fullPath)) {
            $subDirFiles = findAllConfigFilesRecursively($fullPath);
            foreach ($subDirFiles as $subFile) {
                $subFile['relative_path'] = $file . '/' . $subFile['relative_path'];
                $configFiles[] = $subFile;
            }
        }
    }
    
    closedir($handle);
    return $configFiles;
}

class UuidProcessor {
    private $BASE64_KEYS = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=';
    private $BASE64_VALUES = [];
    private $HexChars;
    private $UuidTemplate;
    private $Indices;
    private $uuids; 

    public function __construct() {
        $this->BASE64_VALUES = array_fill(0, 123, 64);
        for ($i = 0; $i < 64; $i++) {
            $this->BASE64_VALUES[ord($this->BASE64_KEYS[$i])] = $i;
        }
        
        $this->HexChars = str_split('0123456789abcdef');
        $this->UuidTemplate = array_fill(0, 36, '');
        $this->UuidTemplate[8] = $this->UuidTemplate[13] = $this->UuidTemplate[18] = $this->UuidTemplate[23] = '-';
        
        $this->Indices = [];
        foreach ($this->UuidTemplate as $idx => $val) {
            if ($val !== '-') $this->Indices[] = $idx;
        }
    }

    public function decodeUuid($base64) {
        if (strlen($base64) !== 22) return $base64;

        $uuid = $this->UuidTemplate;
        $uuid[0] = $base64[0];
        $uuid[1] = $base64[1];
        $pos = 2;

        for ($i = 2; $i < 22; $i += 2) {
            $lhs = $this->BASE64_VALUES[ord($base64[$i])];
            $rhs = $this->BASE64_VALUES[ord($base64[$i+1])];

            $uuid[$this->Indices[$pos++]] = $this->HexChars[$lhs >> 2];
            $uuid[$this->Indices[$pos++]] = $this->HexChars[($lhs & 0x03) << 2 | ($rhs >> 4)];
            $uuid[$this->Indices[$pos++]] = $this->HexChars[$rhs & 0x0F];
        }

        return implode('', $uuid);
    }

    public function process(array $data) {
        $this->uuids = array_map([$this, 'decodeUuid'], $data['uuids']);

        foreach ($data['packs'] as $key => &$pack) {
            $pack = array_map(function($i) { return $this->uuids[$i] ?? $i; }, $pack);
        }

        foreach ($data['versions'] as &$folder) {
            $newFolder = [];
            foreach (array_chunk($folder, 2) as $index => $pair) {
                $uuidIndex = $pair[0] ?? null;
                $value = $pair[1] ?? null;
                
                if (is_numeric($uuidIndex) && isset($this->uuids[$uuidIndex])) {
                    $newFolder[] = [$uuidIndex, $this->uuids[$uuidIndex], $value];
                } else {
                    $newFolder[] = [$uuidIndex, null, $value];
                }
            }
            $folder = $newFolder;
        }

        $redirect = [];
        foreach (array_chunk($data['redirect'], 2) as $pair) {
            if (isset($pair[0], $pair[1], $this->uuids[$pair[0]], $data['bundles'][$pair[1]])) {
                $redirect[] = $this->uuids[$pair[0]];
                $redirect[] = $data['bundles'][$pair[1]];
            }
        }

        return [
            'packs' => $data['packs'],
            'versions' => $data['versions'],
            'redirect' => $redirect
        ];
    }
}

function collectDownloadUrls($basePath, $data, $jsonArray) {
    $urls = [];
    $count = 0;
    
    foreach ($data as $k => $v) {
        if ($k === "import") {
            foreach ($v as $y) {
                $baseName = $y[1];
                $prefix = $y[0];
                $subDir = substr($baseName, 0, 2);
                
                $s_path = "$basePath/$k/$subDir/$baseName.{$y[2]}.json";
                
                // 清理路径
                $s_path = preg_replace('#/+#', '/', $s_path);
                $s_path = ltrim($s_path, '/');
                
                $remoteUrl = "https://file.gugudang.com/res/down/public/p_zhuandaokanshu/web-mobile/lts777/assets/{$s_path}";
                $urls[] = $remoteUrl;
                $count++;
            }
        } else {
            foreach ($v as $y) {
                $baseName = $y[1];
                $prefix = $y[0];
                $subDir = substr($baseName, 0, 2);
                
                // 所有可能的扩展名
                $extensions = [
                    '.bin', '.png', '.jpg', '.jpge', '.astc.br', 
                    '.mp3', '.astc', '.atlas', '.skel', '.webp', 
                    '.bytes', '.json', '.cconb'
                ];
                
                foreach ($extensions as $ext) {
                    $s_path = "$basePath/$k/$subDir/$baseName.{$y[2]}$ext";
                    
                    // 清理路径
                    $s_path = preg_replace('#/+#', '/', $s_path);
                    $s_path = ltrim($s_path, '/');
                    
                    $remoteUrl = "https://file.gugudang.com/res/down/public/p_zhuandaokanshu/web-mobile/lts777/assets/{$s_path}";
                    $urls[] = $remoteUrl;
                    $count++;
                }
            }
        }
    }
    
    return ['urls' => $urls, 'count' => $count];
}

function saveUrlsToFile($urls, $filename = OUTPUT_FILE) {
    // 去重
    $uniqueUrls = array_unique($urls);
    
    // 写入文件
    $result = file_put_contents($filename, implode("\n", $uniqueUrls));
    
    if ($result === false) {
        echo "错误：无法写入文件 {$filename}\n";
        return false;
    }
    
    return [
        'file' => $filename,
        'total_urls' => count($uniqueUrls),
        'duplicates_removed' => count($urls) - count($uniqueUrls)
    ];
}

function showProgress($current, $total, $message = '') {
    $percentage = $total > 0 ? round(($current / $total) * 100, 1) : 0;
    $progressBar = str_repeat('█', floor($percentage / 2)) . str_repeat('░', 50 - floor($percentage / 2));
    
    echo sprintf(
        "\r%s [%s] %d/%d (%.1f%%)",
        $message,
        $progressBar,
        $current,
        $total,
        $percentage
    );
    
    if ($current >= $total) {
        echo "\n";
    }
}

// 主程序开始
echo "========== 资源地址生成器 ==========\n";
echo "输出文件: " . OUTPUT_FILE . "\n";
echo "最大工作进程: " . MAX_WORKERS . "\n\n";

$processor = new UuidProcessor();

// 递归查找所有config文件
echo "正在搜索所有子目录中的config文件...\n";
$configFiles = findAllConfigFilesRecursively(__DIR__);

if (empty($configFiles)) {
    echo "错误：未找到配置文件\n";
    echo "请确保在当前目录或子目录下有 config.*.json 文件\n";
    exit(1);
}

echo "找到 " . count($configFiles) . " 个配置文件\n";

// 显示找到的文件
foreach ($configFiles as $config) {
    echo "  - " . $config['relative_path'] . "\n";
}

echo "\n";

// 统计信息
$totalProcessed = 0;
$allUrls = [];

// 开始处理
echo "开始处理配置文件并生成下载地址...\n";
$startTime = microtime(true);

foreach ($configFiles as $index => $config) {
    showProgress($index + 1, count($configFiles), "处理配置文件中");
    
    $filePath = $config['path'];
    
    // 读取配置文件
    $fileContents = file_get_contents($filePath);
    if ($fileContents === false) {
        continue;
    }
    
    $jsonArray = json_decode($fileContents, true);
    if (json_last_error() !== JSON_ERROR_NONE) {
        continue;
    }
    
    if (!empty($jsonArray['uuids'])) {
        $totalProcessed++;
        
        // 处理UUID
        $uuids = $processor->process($jsonArray);
        
        // 计算config文件所在的目录
        $configDir = dirname($config['relative_path']);
        if ($configDir === '.') {
            $configDir = '';
        }
        
        // 收集下载地址
        $result = collectDownloadUrls($configDir, $uuids['versions'], $jsonArray);
        $allUrls = array_merge($allUrls, $result['urls']);
    }
}

echo "\n所有配置文件处理完成！\n";

// 保存URL到文件
echo "\n正在保存下载地址到文件...\n";
$saveResult = saveUrlsToFile($allUrls);

if ($saveResult) {
    $endTime = microtime(true);
    $elapsedTime = round($endTime - $startTime, 2);
    
    echo "\n========== 处理完成 ==========\n";
    echo "处理的配置文件: {$totalProcessed}/" . count($configFiles) . "\n";
    echo "生成的URL总数: " . $saveResult['total_urls'] . "\n";
    echo "移除的重复URL: " . $saveResult['duplicates_removed'] . "\n";
    echo "耗时: {$elapsedTime}秒\n";
    echo "输出文件: " . $saveResult['file'] . "\n";
    
    // 生成wget命令
    echo "\n========== 下载命令wget -r -i 1.txt --no-check-certificate==========\n";
    echo "您可以使用以下命令下载所有文件：\n\n";
    echo "方法1：使用wget（推荐）\n";
    echo "wget -c -i " . OUTPUT_FILE . " --timeout=30 --tries=3 --random-wait --no-check-certificate \\\n";
    echo "     --user-agent='Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36' \\\n";
    echo "     --directory-prefix=v1 --force-directories\n\n";
    
    echo "方法2：使用aria2c（更快）\n";
    echo "aria2c -c -i " . OUTPUT_FILE . " --max-concurrent-downloads=" . MAX_WORKERS . " \\\n";
    echo "       --timeout=30 --max-tries=3 --check-certificate=false \\\n";
    echo "       --user-agent='Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36' \\\n";
    echo "       --dir=v1 --continue\n\n";
    
    echo "方法3：使用xargs并行下载\n";
    echo "cat " . OUTPUT_FILE . " | xargs -P " . MAX_WORKERS . " -I {} wget -c {} --timeout=30 --tries=3 \\\n";
    echo "  --no-check-certificate --directory-prefix=v1 --force-directories\n\n";
    
    echo "温馨提示：\n";
    echo "1. 如果下载中断，可以使用相同的命令继续下载（wget的-c参数支持断点续传）\n";
    echo "2. 建议在服务器或网络环境较好的地方执行下载\n";
    echo "3. 下载的文件将保存在 v1 目录中\n";
    echo "4. 您可以根据需要调整并发数和超时时间\n";
    
    // 检查文件大小
    $fileSize = filesize(OUTPUT_FILE);
    if ($fileSize > 0) {
        $sizeKB = round($fileSize / 1024, 2);
        $sizeMB = round($fileSize / (1024 * 1024), 2);
        echo "\nURL列表文件大小: {$sizeKB} KB ({$sizeMB} MB)\n";
    }
} else {
    echo "错误：保存URL列表失败\n";
}
?>