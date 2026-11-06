import http from 'node:http';
import fs from 'node:fs';
import path from 'node:path';
import { fileURLToPath } from 'node:url';

const root = path.join(path.dirname(fileURLToPath(import.meta.url)), 'public');
const port = Number(process.env.PORT || 3000);
const mime = {
  '.html': 'text/html; charset=utf-8',
  '.css': 'text/css; charset=utf-8',
  '.js': 'text/javascript; charset=utf-8',
  '.json': 'application/json; charset=utf-8',
  '.svg': 'image/svg+xml',
  '.png': 'image/png',
  '.jpg': 'image/jpeg',
  '.webp': 'image/webp',
};

function sendJson(res, status, data) {
  res.writeHead(status, { 'Content-Type': 'application/json; charset=utf-8', 'Cache-Control': 'no-store' });
  res.end(JSON.stringify(data));
}

async function readJson(req) {
  let body = '';
  for await (const chunk of req) {
    body += chunk;
    if (body.length > 12000) throw new Error('request_too_large');
  }
  return JSON.parse(body || '{}');
}

async function officialContext(questionText) {
  const needsSource = /(موعد|مواعيد|رسوم|تسجيل|تسجيل|آلية|نتيجة|صلاحية|فرصة|اختبار ورقي|محوسب|رسمي|هيئة|قياس)/i.test(questionText);
  if (!needsSource) return { text: '', sources: [] };
  const googleUrl = `https://www.google.com/search?q=${encodeURIComponent(`site:etec.gov.sa ${questionText}`)}`;
  const sources = ['https://etec.gov.sa/', googleUrl];
  try {
    const controller = new AbortController(); const timeout = setTimeout(() => controller.abort(), 3500);
    const response = await fetch('https://etec.gov.sa/', { signal: controller.signal, headers: { 'User-Agent': 'OfqStudyAssistant/# ofok