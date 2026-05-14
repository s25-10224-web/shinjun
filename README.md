# shinjun
import React, { useState, useEffect } from 'react';
import { 
  Lock, 
  User, 
  FileText, 
  BookOpen, 
  AlertTriangle, 
  CheckCircle, 
  ExternalLink, 
  Lightbulb, 
  ArrowRight, 
  LogOut, 
  ChevronRight, 
  Info, 
  Target, 
  GraduationCap, 
  TrendingUp, 
  BarChart3, 
  School, 
  Sparkles, 
  Search, 
  MessageSquareQuote, 
  ShieldCheck, 
  Zap, 
  Globe, 
  Languages, 
  ClipboardList 
} from 'lucide-react';

// --- 설정 및 상수 ---
const APP_NAME = "ADMIT AI";
const LOGIN_CREDENTIALS = { id: 'admin', pw: '1234' };
const apiKey = ""; 

const FORBIDDEN_WORDS = ["토익", "토플", "텝스", "경시대회", "금상", "부모님", "직업", "교수", "해외연수"];

const MAJORS_BY_FIELD = {
  "공학계열": ["컴퓨터공학", "AI공학", "기계공학", "전기전자", "화학공학", "데이터사이언스"],
  "의학계열": ["의예과", "치의예", "약학", "간호학", "수의예"],
  "인문/사회": ["경영학", "경제학", "심리학", "정치외교", "미디어학", "교육학"],
  "자연과학": ["물리학", "생명과학", "수학", "천문학"]
};

export default function App() {
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  const [loginId, setLoginId] = useState('');
  const [loginPw, setLoginPw] = useState('');
  const [activeTab, setActiveTab] = useState('editor');
  
  // 데이터 상태
  const [content, setContent] = useState('');
  const [targetField, setTargetField] = useState('');
  const [targetMajor, setTargetMajor] = useState('');
  const [gpa, setGpa] = useState('');
  const [mockScore, setMockScore] = useState('');
  const [engScore, setEngScore] = useState('');
  
  const [analysisResult, setAnalysisResult] = useState(null);
  const [isAnalyzing, setIsAnalyzing] = useState(false);
  const [errorMsg, setErrorMsg] = useState('');

  const handleLogin = (e) => {
    e.preventDefault();
    if (loginId === LOGIN_CREDENTIALS.id && loginPw === LOGIN_CREDENTIALS.pw) {
      setIsLoggedIn(true);
      setErrorMsg('');
    } else {
      setErrorMsg('아이디 또는 비밀번호가 올바르지 않습니다.');
    }
  };

  const analyzeContent = async () => {
    if (!content.trim()) return;
    setIsAnalyzing(true);
    setErrorMsg('');

    try {
      const systemPrompt = `
        당신은 한국 입시와 글로벌 유학 컨설팅 전문가입니다. 
        사용자의 데이터(내신:${gpa}, 모의고사:${mockScore}, 희망학과:${targetMajor})를 분석하여 한국어 중심의 JSON을 반환하세요.

        응답은 반드시 다음 구조를 가진 JSON 객체여야 합니다:
        {
          "suggestions": [{"original": "...", "improved": "...", "reason": "..."}],
          "alignment": {"score": 85, "analysis": "...", "aiRecommendations": ["탐구주제1", "탐구주제2", "탐구주제3"]},
          "domesticPlan": {
            "challenge": {"name": "대학명", "type": "전형", "strategy": "전략"},
            "target": {"name": "대학명", "type": "전형", "strategy": "전략"},
            "safety": {"name": "대학명", "type": "전형", "strategy": "전략"}
          },
          "globalPlan": {
            "challenge": {"name": "대학명", "location": "국가", "tuition": "학비"},
            "target": {"name": "대학명", "location": "국가", "tuition": "학비"},
            "safety": {"name": "대학명", "location": "국가", "tuition": "학비"}
          },
          "globalRequirements": ["준비사항1", "준비사항2", "준비사항3"],
          "roadmap": ["1-2월 활동", "3-4월 활동"]
        }
      `;
      
      const userQuery = `내용: "${content}", 학과: "${targetMajor}", 성적: "내신 ${gpa}, 모평 ${mockScore}", 영어성적: "${engScore}"`;

      const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          contents: [{ parts: [{ text: userQuery }] }],
          systemInstruction: { parts: [{ text: systemPrompt }] },
          generationConfig: { 
            responseMimeType: "application/json",
            temperature: 0.7
          }
        })
      });

      if (!response.ok) throw new Error('API 호출에 실패했습니다.');

      const data = await response.json();
      const resultText = data.candidates?.[0]?.content?.parts?.[0]?.text;
      
      if (!resultText) throw new Error('응답 데이터가 비어있습니다.');
      
      const parsed = JSON.parse(resultText);
      setAnalysisResult(parsed);
    } catch (err) {
      console.error(err);
      setErrorMsg("분석 중 오류가 발생했습니다. 성적 정보와 입력 내용을 다시 확인해 주세요.");
    } finally {
      setIsAnalyzing(false);
    }
  };

  if (!isLoggedIn) {
    return (
      <div className="min-h-screen bg-slate-50 flex items-center justify-center p-4">
        <div className="max-w-md w-full bg-white rounded-3xl shadow-2xl p-10 space-y-8 border border-slate-100">
          <div className="text-center space-y-3">
            <div className="inline-flex p-4 bg-indigo-600 rounded-2xl text-white shadow-xl shadow-indigo-100">
              <Sparkles size={32} />
            </div>
            <h1 className="text-3xl font-black text-slate-900 tracking-tighter">ADMIT AI</h1>
            <p className="text-slate-500 text-sm font-bold">국내외 통합 입시 컨설팅 시스템</p>
          </div>
          <form onSubmit={handleLogin} className="space-y-4">
            <input 
              type="text" className="w-full px-5 py-4 bg-slate-50 border border-slate-200 rounded-2xl outline-none focus:ring-2 focus:ring-indigo-500 font-bold" 
              placeholder="아이디 (admin)" value={loginId} onChange={(e) => setLoginId(e.target.value)}
            />
            <input 
              type="password" className="w-full px-5 py-4 bg-slate-50 border border-slate-200 rounded-2xl outline-none focus:ring-2 focus:ring-indigo-500 font-bold" 
              placeholder="비밀번호 (1234)" value={loginPw} onChange={(e) => setLoginPw(e.target.value)}
            />
            {errorMsg && <p className="text-red-500 text-xs font-bold text-center">{errorMsg}</p>}
            <button type="submit" className="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-black py-5 rounded-2xl transition-all shadow-xl shadow-indigo-100">분석 시스템 시작</button>
          </form>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-slate-50 flex flex-col text-slate-800">
      <header className="bg-white/80 backdrop-blur-md border-b border-slate-200 sticky top-0 z-50 px-8 py-4 flex items-center justify-between">
        <div className="flex items-center gap-3">
          <div className="bg-indigo-600 p-2 rounded-lg text-white"><Globe size={20} /></div>
          <h2 className="text-xl font-black tracking-tighter">ADMIT <span className="text-indigo-600">AI GLOBAL</span></h2>
        </div>
        <div className="flex items-center gap-4 text-center">
          <button onClick={() => setActiveTab('editor')} className={`px-4 py-2 rounded-xl text-sm font-black transition-all ${activeTab === 'editor' ? 'bg-indigo-600 text-white' : 'text-slate-400 hover:bg-slate-100'}`}>입시 분석기</button>
          <button onClick={() => setActiveTab('profile')} className={`px-4 py-2 rounded-xl text-sm font-black transition-all ${activeTab === 'profile' ? 'bg-indigo-600 text-white' : 'text-slate-400 hover:bg-slate-100'}`}>성적 및 진로설정</button>
          <div className="h-6 w-px bg-slate-200 mx-2"></div>
          <button onClick={() => setIsLoggedIn(false)} className="text-slate-400 hover:text-red-500 text-xs font-black uppercase">로그아웃</button>
        </div>
      </header>

      <main className="flex-1 p-8 lg:p-12 overflow-y-auto">
        {activeTab === 'editor' ? (
          <div className="max-w-6xl mx-auto space-y-12">
            <section className="flex flex-col md:flex-row md:items-end justify-between gap-4">
              <div>
                <h3 className="text-3xl font-black text-slate-900 tracking-tighter">통합 생기부 에디터</h3>
                <p className="text-slate-500 font-medium italic">입력하신 데이터는 Gemini AI가 보안 처리 후 실시간 분석합니다.</p>
              </div>
              <div className="flex gap-2">
                <span className="px-4 py-2 bg-white rounded-full border text-[11px] font-black text-indigo-600 shadow-sm">🎯 희망학과: {targetMajor || '미설정'}</span>
                <span className="px-4 py-2 bg-white rounded-full border text-[11px] font-black text-emerald-600 shadow-sm">📊 내신: {gpa || '-'}등급</span>
              </div>
            </section>

            <div className="bg-white rounded-[2.5rem] shadow-2xl shadow-slate-200 border border-slate-200 overflow-hidden">
              <textarea 
                className="w-full p-10 h-80 outline-none text-slate-700 leading-relaxed resize-none text-xl placeholder:text-slate-200 font-medium"
                placeholder="학생부 내용을 자유롭게 입력하세요. (교과세특, 자율활동 등)"
                value={content}
                onChange={(e) => setContent(e.target.value)}
              />
              <div className="px-10 py-6 bg-slate-50 border-t border-slate-100 flex justify-between items-center">
                <p className="text-xs font-bold text-slate-400">현재 글자 수: {content.length}자</p>
                <button 
                  onClick={analyzeContent}
                  disabled={isAnalyzing || !content.trim() || !targetMajor}
                  className="bg-indigo-600 hover:bg-indigo-700 text-white px-10 py-4 rounded-2xl font-black flex items-center gap-3 transition-all disabled:opacity-50"
                >
                  {isAnalyzing ? <Zap size={18} className="animate-spin" /> : <Sparkles size={18} />}
                  {isAnalyzing ? "분석 엔진 가동 중..." : "AI 정밀 분석 시작"} <ArrowRight size={18} />
                </button>
              </div>
            </div>

            {analysisResult && (
              <div className="grid grid-cols-1 lg:grid-cols-12 gap-8 animate-in fade-in slide-in-from-bottom-10 duration-700">
                <div className="lg:col-span-7 space-y-8">
                  <div className="bg-white p-8 rounded-3xl border border-slate-200 shadow-sm">
                    <h4 className="text-xl font-black mb-6 flex items-center gap-2"><Sparkles className="text-indigo-600" /> AI 문장 고도화 제안</h4>
                    <div className="space-y-6">
                      {(analysisResult.suggestions || []).map((s, i) => (
                        <div key={i} className="p-5 bg-slate-50 rounded-2xl border border-slate-100">
                          <p className="text-xs text-slate-400 mb-2 font-bold italic line-through">기존: "{s.original}"</p>
                          <p className="text-base font-black text-indigo-900 mb-2">변경: "{s.improved}"</p>
                          <p className="text-[11px] text-slate-500 font-bold bg-white p-2 rounded-lg inline-block border border-slate-100">이유: {s.reason}</p>
                        </div>
                      ))}
                    </div>
                  </div>

                  <div className="bg-white p-8 rounded-3xl border border-slate-200 shadow-sm">
                    <h4 className="text-xl font-black mb-6 flex items-center gap-2"><Target className="text-red-500" /> 전공 적합성 분석</h4>
                    <div className="flex items-center gap-6 mb-6">
                      <div className="text-4xl font-black text-indigo-600">{analysisResult.alignment?.score || 0}%</div>
                      <div className="h-2 flex-1 bg-slate-100 rounded-full overflow-hidden">
                        <div className="h-full bg-indigo-600 transition-all duration-1000" style={{width: `${analysisResult.alignment?.score || 0}%`}}></div>
                      </div>
                    </div>
                    <div className="grid grid-cols-1 gap-3">
                      <p className="text-xs font-black text-slate-400 uppercase tracking-widest">추천 심화 탐구 과제</p>
                      {(analysisResult.alignment?.aiRecommendations || []).map((rec, i) => (
                        <div key={i} className="p-4 bg-indigo-50 border border-indigo-100 rounded-xl text-xs font-bold text-indigo-900">✨ {rec}</div>
                      ))}
                    </div>
                  </div>
                </div>

                <div className="lg:col-span-5 space-y-8">
                  <div className="bg-white p-8 rounded-3xl border border-slate-200 shadow-xl">
                    <h4 className="text-xl font-black mb-6 flex items-center gap-2"><School className="text-indigo-600" /> 국내 대학 라인업</h4>
                    <div className="space-y-4">
                      <RecommendItem label="상향" data={analysisResult.domesticPlan?.challenge} color="amber" />
                      <RecommendItem label="적정" data={analysisResult.domesticPlan?.target} color="indigo" />
                      <RecommendItem label="안정" data={analysisResult.domesticPlan?.safety} color="emerald" />
                    </div>
                  </div>

                  <div className="bg-indigo-900 p-8 rounded-3xl shadow-2xl text-white">
                    <h4 className="text-xl font-black mb-6 flex items-center gap-2"><Globe className="text-indigo-400" /> GLOBAL 해외 명문대</h4>
                    <div className="space-y-4 mb-8">
                      <RecommendItemGlobal label="상향" data={analysisResult.globalPlan?.challenge} />
                      <RecommendItemGlobal label="적정" data={analysisResult.globalPlan?.target} />
                      <RecommendItemGlobal label="안정" data={analysisResult.globalPlan?.safety} />
                    </div>
                    
                    <div className="bg-white/10 p-5 rounded-2xl border border-white/20 space-y-4">
                      <p className="text-[11px] font-black text-indigo-300 flex items-center gap-2"><Languages size={14} /> 유학 필수 준비 항목</p>
                      <div className="space-y-3">
                        {(analysisResult.globalRequirements || []).map((req, i) => (
                          <div key={i} className="flex gap-2 items-start">
                            <CheckCircle size={14} className="mt-0.5 text-indigo-400 flex-shrink-0" />
                            <p className="text-xs font-bold leading-relaxed">{req}</p>
                          </div>
                        ))}
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            )}
          </div>
        ) : (
          <div className="max-w-4xl mx-auto space-y-10 animate-in fade-in slide-in-from-right-10">
            <h3 className="text-3xl font-black text-slate-900 tracking-tighter text-center">성적 및 진로 데이터 설정</h3>
            <div className="grid grid-cols-1 md:grid-cols-2 gap-8">
              <div className="bg-white p-8 rounded-3xl border border-slate-200 space-y-6">
                <h4 className="font-black text-lg flex items-center gap-2 text-indigo-600"><Target size={20} /> 희망 진로 설정</h4>
                <div className="space-y-4">
                  <select className="w-full p-4 bg-slate-50 border rounded-2xl font-bold" value={targetField} onChange={(e) => setTargetField(e.target.value)}>
                    <option value="">계열 선택</option>
                    {Object.keys(MAJORS_BY_FIELD).map(f => <option key={f} value={f}>{f}</option>)}
                  </select>
                  <select className="w-full p-4 bg-slate-50 border rounded-2xl font-bold" disabled={!targetField} value={targetMajor} onChange={(e) => setTargetMajor(e.target.value)}>
                    <option value="">학과 선택</option>
                    {targetField && MAJORS_BY_FIELD[targetField].map(m => <option key={m} value={m}>{m}</option>)}
                  </select>
                </div>
              </div>
              <div className="bg-white p-8 rounded-3xl border border-slate-200 space-y-6">
                <h4 className="font-black text-lg flex items-center gap-2 text-emerald-600"><TrendingUp size={20} /> 현재 성적 지표</h4>
                <div className="space-y-4">
                  <input type="number" step="0.1" placeholder="내신 평균 등급 (예: 1.5)" className="w-full p-4 bg-slate-50 border rounded-2xl font-bold" value={gpa} onChange={(e) => setGpa(e.target.value)} />
                  <input type="number" placeholder="모평 백분위 (예: 96)" className="w-full p-4 bg-slate-50 border rounded-2xl font-bold" value={mockScore} onChange={(e) => setMockScore(e.target.value)} />
                  <input type="text" placeholder="공인영어성적 (예: TOEFL 105)" className="w-full p-4 bg-slate-50 border rounded-2xl font-bold" value={engScore} onChange={(e) => setEngScore(e.target.value)} />
                </div>
              </div>
            </div>
            <button onClick={() => setActiveTab('editor')} className="w-full bg-indigo-600 text-white py-5 rounded-3xl font-black shadow-2xl hover:scale-[1.01] transition-all">설정 완료 및 분석 시작</button>
          </div>
        )}
      </main>
    </div>
  );
}

function RecommendItem({ label, data, color }) {
  if (!data) return null;
  const colorMap = {
    amber: 'border-amber-100 bg-amber-50 text-amber-700',
    indigo: 'border-indigo-100 bg-indigo-50 text-indigo-700',
    emerald: 'border-emerald-100 bg-emerald-50 text-emerald-700'
  };
  return (
    <div className={`p-4 rounded-2xl border ${colorMap[color]}`}>
      <div className="flex justify-between items-center mb-1">
        <span className="text-[10px] font-black uppercase tracking-widest">{label}</span>
        <span className="text-[9px] font-bold px-2 py-0.5 bg-white rounded-md">{data.type || '종합/일반'}</span>
      </div>
      <p className="text-sm font-black text-slate-900">{data.name || '정보 없음'}</p>
      <p className="text-[10px] font-bold opacity-70 mt-1">{data.strategy || '성적과 생기부의 조화가 필요한 단계'}</p>
    </div>
  );
}

function RecommendItemGlobal({ label, data }) {
  if (!data) return null;
  return (
    <div className="p-4 rounded-2xl bg-white/5 border border-white/10 hover:bg-white/10 transition-colors group">
      <div className="flex justify-between items-center mb-1">
        <span className="text-[9px] font-black text-indigo-400 uppercase tracking-widest">{label}</span>
        <span className="text-[9px] font-bold opacity-50">{data.location || 'Global'}</span>
      </div>
      <p className="text-sm font-black text-white group-hover:text-indigo-300 transition-colors">{data.name || 'University'}</p>
      <p className="text-[10px] font-bold text-indigo-200/60 mt-1">예상 학비: {data.tuition || '정보 없음'}</p>
    </div>
  );
}
