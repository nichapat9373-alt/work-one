import React, { useState, useEffect, useRef } from 'react';
import { initializeApp } from 'firebase/app';
import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from 'firebase/auth';
import { 
  getFirestore, 
  doc, 
  onSnapshot, 
  setDoc,
  enableNetwork,
  disableNetwork
} from 'firebase/firestore';
import { 
  Save, 
  Lock, 
  ChevronRight,
  Stethoscope as NurseIcon,
  Upload,
  Image as ImageIcon,
  CheckCircle2,
  ChevronLeft,
  GraduationCap,
  BookOpen,
  Award,
  User as UserIcon,
  Phone,
  Mail,
  MapPin,
  Heart,
  Star,
  WifiOff
} from 'lucide-react';

// --- การตั้งค่า Firebase ---
const firebaseConfig = JSON.parse(__firebase_config);
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
const appId = typeof __app_id !== 'undefined' ? __app_id : 'nursing-port-2570';

// --- ระบบดึงรูปภาพสำรอง ---
const getPlaceholder = (type) => {
  const images = {
    profile: "https://images.unsplash.com/photo-1573496359142-b8d87734a5a2?auto=format&fit=crop&q=80&w=800",
    nurse_activity: "https://images.unsplash.com/photo-1576091160550-217359f4ecf8?auto=format&fit=crop&q=80&w=800",
    volunteer: "https://images.unsplash.com/photo-1559027615-cd2673555936?auto=format&fit=crop&q=80&w=800",
    certificate: "https://images.unsplash.com/photo-1589330694653-ded6df03f754?auto=format&fit=crop&q=80&w=800",
    leadership: "https://images.unsplash.com/photo-1552664730-d307ca884978?auto=format&fit=crop&q=80&w=800"
  };
  return images[type] || images.nurse_activity;
};

// --- ระบบบีบอัดรูปภาพ ---
const compressImage = (base64Str, maxWidth = 800, maxHeight = 800) => {
  return new Promise((resolve) => {
    const img = new Image();
    img.src = base64Str;
    img.onerror = () => resolve(getPlaceholder('nurse_activity')); 
    img.onload = () => {
      const canvas = document.createElement('canvas');
      let width = img.width;
      let height = img.height;
      if (width > height) {
        if (width > maxWidth) { height *= maxWidth / width; width = maxWidth; }
      } else {
        if (height > maxHeight) { width *= maxHeight / height; height = maxHeight; }
      }
      canvas.width = width;
      canvas.height = height;
      const ctx = canvas.getContext('2d');
      ctx.drawImage(img, 0, 0, width, height);
      resolve(canvas.toDataURL('image/jpeg', 0.6));
    };
  });
};

const EditableText = ({ value, onChange, className, isAdmin, multiline = false }) => {
  if (!isAdmin) return <span className={className}>{value || "-"}</span>;
  return multiline ? (
    <textarea value={value} onChange={(e) => onChange(e.target.value)} className={`bg-blue-50/50 border-b-2 border-blue-400 outline-none w-full p-2 rounded ${className}`} />
  ) : (
    <input type="text" value={value} onChange={(e) => onChange(e.target.value)} className={`bg-blue-50/50 border-b-2 border-blue-400 outline-none p-1 rounded ${className}`} />
  );
};

const ImageUploader = ({ src, className, onImageChange, isAdmin, type = "nurse_activity" }) => {
  const fileInputRef = useRef(null);
  const displayImage = (src && src.trim() !== "") ? src : getPlaceholder(type);
  const [isCompressing, setIsCompressing] = useState(false);

  const handleUpload = async (e) => {
    const file = e.target.files[0];
    if (!file) return;
    setIsCompressing(true);
    const reader = new FileReader();
    reader.onload = async (event) => {
      const result = event.target.result;
      const compressed = await compressImage(result);
      onImageChange(compressed);
      setIsCompressing(false);
    };
    reader.readAsDataURL(file);
  };

  return (
    <div className={`relative group transition-all overflow-hidden bg-slate-200 ${className}`} onClick={() => isAdmin && !isCompressing && fileInputRef.current?.click()}>
      {isCompressing && <div className="absolute inset-0 z-20 bg-blue-900/60 flex flex-col items-center justify-center text-white text-[10px] font-bold"><NurseIcon className="animate-spin mb-1"/> กำลังประมวลผล...</div>}
      <img src={displayImage} className="w-full h-full object-cover" alt="Portfolio" onError={(e) => e.target.src = getPlaceholder(type)} />
      {isAdmin && (
        <div className="absolute inset-0 bg-blue-600/20 opacity-0 group-hover:opacity-100 flex items-center justify-center transition-opacity cursor-pointer">
          <div className="bg-white p-3 rounded-full shadow-xl"><Upload className="text-blue-600" size={24}/></div>
        </div>
      )}
      <input type="file" ref={fileInputRef} onChange={handleUpload} accept="image/*" className="hidden" />
    </div>
  );
};

const App = () => {
  const [user, setUser] = useState(null);
  const [isAdmin, setIsAdmin] = useState(false);
  const [password, setPassword] = useState('');
  const [activePage, setActivePage] = useState(0);
  const [loading, setLoading] = useState(true);
  const [showLogin, setShowLogin] = useState(false);
  const [isSaving, setIsSaving] = useState(false);
  const [connectionError, setConnectionError] = useState(false);
  
  const [portData, setPortData] = useState({
    profile: { 
      name: "ณิชาภัทร ฤทธิ์ทรงเมือง", 
      nickname: "น้องแก้ม", school: "โรงเรียนนารีนุกูล", motto: "พยาบาลด้วยหัวใจ ดูแลด้วยหลักวิชาการ",
      gpax: "3.95", birthDate: "13 มิถุนายน 2553", bloodType: "O", address: "อ.เมือง จ.อุบลราชธานี 34000",
      contact: { tel: "08x-xxx-xxxx", email: "nichaphat.r@email.com", line: "neoy_nurse" },
      skills: ["การปฐมพยาบาลเบื้องต้น (BLS)", "การทำงานภายใต้สภาวะกดดัน", "การสื่อสารภาษอังกฤษ"],
      mainPhoto: "", profilePhoto: "",
      education: [
        { level: "ประถมศึกษา", school: "โรงเรียนอนุบาลอุบลราชธานี", year: "2559-2564", gpax: "4.00" },
        { level: "มัธยมศึกษาตอนต้น", school: "โรงเรียนนารีนุกูล", year: "2565-2567", gpax: "3.98" },
        { level: "มัธยมศึกษาตอนปลาย", school: "โรงเรียนนารีนุกูล (วิทย์-คณิต)", year: "2568-ปัจจุบัน", gpax: "3.95" }
      ],
      sop_content: "เหตุผลที่ข้าพเจ้าปรารถนาจะเข้าศึกษาต่อในคณะพยาบาลศาสตร์ เนื่องจากข้าพเจ้ามีความเชื่อมั่นว่าวิชาชีพพยาบาลเป็นวิชาชีพที่มีเกียรติและได้ช่วยเหลือผู้อื่นอย่างใกล้ชิด ประสบการณ์จากการดูแลคนใกล้ชิดยามเจ็บป่วยทำให้ข้าพเจ้าได้เห็นความสำคัญของการเอาใจใส่และการดูแลที่ถูกต้องตามหลักวิชาการ ข้าพเจ้ามีความอดทน มีจิตอาสา และพร้อมที่จะเรียนรู้เพื่อพัฒนาตนเองให้เป็นพยาบาลวิชาชีพที่มีคุณภาพ เพื่อกลับไปดูแลสุขภาพของคนในชุมชนและสังคมสืบต่อไป",
      academic_activities: [
        { id: 1, title: "ค่ายสานฝันพยาบาลสภากาชาดไทย", desc: "เรียนรู้ทักษะพื้นฐานทางการพยาบาลและการเตรียมความพร้อม", imageUrl: "" },
        { id: 2, title: "โครงงานวิทยาศาสตร์ระดับภาค", desc: "ศึกษาการยับยั้งเชื้อแบคทีเรียด้วยสมุนไพรพื้นถิ่น", imageUrl: "" }
      ],
      volunteer_activities: [
        { id: 1, title: "หน่วยอาสาสมัครปฐมพยาบาล", desc: "ปฏิบัติหน้าที่คัดกรองและปฐมพยาบาลเบื้องต้นในงานกีฬาสี", imageUrl: "" },
        { id: 2, title: "จิตอาสาพัฒนาโรงพยาบาล", desc: "ช่วยเหลือเจ้าหน้าที่ในส่วนงานลงทะเบียนและนำทางผู้ป่วย", imageUrl: "" }
      ],
      leadership_activities: [
        { id: 1, title: "ประธานชมรมยุวกาชาด", desc: "วางแผนและจัดกิจกรรมรณรงค์ด้านสุขภาพในโรงเรียน", imageUrl: "" },
        { id: 2, title: "กรรมการสภานักเรียน", desc: "ประสานงานระหว่างนักเรียนและฝ่ายบริหารโรงเรียน", imageUrl: "" }
      ],
      certificates_high: [
        { id: 1, title: "รางวัลชนะเลิศโครงงานวิทยาศาสตร์", issuer: "วว.", imageUrl: "" },
        { id: 2, title: "เกียรติบัตรเรียนดีเด่น 4.00", issuer: "ร.ร.นารีนุกูล", imageUrl: "" }
      ],
      certificates_moral: [
        { id: 1, title: "เยาวชนต้นแบบด้านจริยธรรม", issuer: "กระทรวงศึกษาธิการ", imageUrl: "" }
      ],
      certificates_others: [
        { id: 1, title: "Basic Life Support Training", issuer: "Thai Red Cross", imageUrl: "" },
        { id: 2, title: "IELTS Score 6.5", issuer: "British Council", imageUrl: "" }
      ]
    }
  });

  // Rule 3: Auth Before Queries
  useEffect(() => {
    const initAuth = async () => {
      try {
        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
          await signInWithCustomToken(auth, __initial_auth_token);
        } else {
          await signInAnonymously(auth);
        }
      } catch (err) { 
        console.error("Auth Error:", err); 
        setConnectionError(true);
      }
    };
    initAuth();
    const unsubscribe = onAuthStateChanged(auth, (u) => {
      setUser(u);
      if (u) setConnectionError(false);
    });
    return () => unsubscribe();
  }, []);

  // Firestore Sync with Connection Management
  useEffect(() => {
    if (!user) return;
    
    setLoading(true);
    const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'content', 'portfolio_main');
    
    // Attempt to keep Firestore online
    enableNetwork(db).catch(() => {});

    const unsubscribe = onSnapshot(docRef, 
      (docSnap) => {
        if (docSnap.exists()) {
          const data = docSnap.data();
          if (data && data.profile) {
            setPortData(prev => ({
              ...prev,
              ...data,
              profile: { ...prev.profile, ...data.profile }
            }));
          }
        }
        setLoading(false);
        setConnectionError(false);
      }, 
      (error) => {
        console.error("Firestore Listen Error:", error);
        // Don't set loading false immediately to allow retries
        if (error.code === 'unavailable') {
          setConnectionError(true);
        }
        setLoading(false);
      }
    );

    return () => unsubscribe();
  }, [user]);

  const saveData = async () => {
    if (!user || !isAdmin) return;
    setIsSaving(true);
    try {
      const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'content', 'portfolio_main');
      await setDoc(docRef, portData);
      alert("บันทึกข้อมูลเรียบร้อยแล้ว!");
    } catch (err) { 
      console.error("Save Error:", err);
      alert("ไม่สามารถบันทึกได้ กรุณาตรวจสอบการเชื่อมต่ออินเทอร์เน็ต"); 
    }
    finally { setIsSaving(false); }
  };

  const updateField = (path, val) => {
    const newData = JSON.parse(JSON.stringify(portData));
    const keys = path.split('.');
    let current = newData;
    for (let i = 0; i < keys.length - 1; i++) current = current[keys[i]];
    current[keys[keys.length - 1]] = val;
    setPortData(newData);
  };

  const updateItem = (cat, id, field, val) => {
    const newData = JSON.parse(JSON.stringify(portData));
    const idx = newData.profile[cat].findIndex(x => x.id === id);
    if (idx > -1) { newData.profile[cat][idx][field] = val; setPortData(newData); }
  };

  const pages = [
    { title: "หน้าปก" }, { title: "ประวัติส่วนตัว" }, { title: "ประวัติการศึกษา" },
    { title: "เรียงความ (SOP)" }, { title: "กิจกรรมวิชาการ" }, { title: "กิจกรรมจิตอาสา" }, 
    { title: "กิจกรรมผู้นำ" }, { title: "เกียรติบัตร 1" }, { title: "เกียรติบัตร 2" }, { title: "เกียรติบัตรอื่นๆ" }
  ];

  if (loading && !connectionError) return <div className="h-screen flex items-center justify-center bg-white"><NurseIcon className="animate-spin text-blue-600" size={48} /></div>;

  return (
    <div className="min-h-screen bg-slate-50 flex flex-col font-sans text-slate-900 overflow-x-hidden">
      {/* Connection Error Notification */}
      {connectionError && (
        <div className="bg-red-600 text-white text-[10px] py-1 px-4 flex items-center justify-center gap-2 font-bold uppercase tracking-widest z-[200]">
          <WifiOff size={14}/> กำลังทำงานในโหมดออฟไลน์... กรุณาตรวจสอบการเชื่อมต่ออินเทอร์เน็ต
        </div>
      )}

      <header className="bg-blue-950 text-white p-4 flex justify-between items-center shadow-xl sticky top-0 z-[100]">
        <div className="flex items-center gap-2">
          <NurseIcon size={24} className="text-blue-400" />
          <h1 className="font-black text-lg tracking-tighter uppercase italic">Nurse<span className="text-blue-400 italic">Portfolio</span></h1>
        </div>
        <div className="flex gap-2">
           {!isAdmin ? (
             <button onClick={() => setShowLogin(true)} className="bg-blue-800 text-[10px] px-4 py-2 rounded-full font-bold uppercase tracking-widest"><Lock size={12} className="inline mr-1"/> ปลดล็อก</button>
           ) : (
             <button onClick={saveData} disabled={isSaving} className={`${isSaving ? 'bg-slate-500' : 'bg-green-600'} text-[10px] px-6 py-2 rounded-full font-black uppercase tracking-widest flex items-center gap-2 shadow-lg`}>
               {isSaving ? "กำลังบันทึก..." : <><Save size={12}/> บันทึกข้อมูล</>}
             </button>
           )}
        </div>
      </header>

      <div className="flex-1 flex flex-col lg:flex-row p-4 gap-6 max-w-7xl mx-auto w-full">
        <aside className="lg:w-56 shrink-0 overflow-x-auto lg:overflow-visible py-2">
          <div className="flex lg:flex-col gap-2">
            {pages.map((p, i) => (
              <button key={i} onClick={() => setActivePage(i)} className={`whitespace-nowrap text-left p-3 rounded-2xl text-[10px] font-black transition-all flex items-center gap-3 ${activePage === i ? 'bg-blue-600 text-white shadow-lg scale-105' : 'bg-white text-slate-400 border border-slate-200 hover:border-blue-300'}`}>
                <span className={`w-5 h-5 rounded-lg flex items-center justify-center ${activePage === i ? 'bg-white/20 text-white' : 'bg-slate-100 text-slate-400'}`}>{i+1}</span>
                {p.title}
              </button>
            ))}
          </div>
        </aside>

        <main className="flex-1 bg-white rounded-[3rem] shadow-2xl min-h-[850px] border-[12px] border-blue-950 p-6 md:p-12 relative flex flex-col transition-all">
          
          {/* หน้า 0: หน้าปก */}
          {activePage === 0 && (
            <div className="h-full flex flex-col items-center justify-center text-center animate-in fade-in duration-500">
               <div className="bg-blue-950 text-white px-8 py-2 rounded-full font-black text-xs tracking-widest mb-10 shadow-xl uppercase italic">The Professional Portfolio</div>
               <EditableText value={portData.profile.name} onChange={(v) => updateField('profile.name', v)} className="text-5xl font-black text-blue-950 mb-2 uppercase italic tracking-tighter" isAdmin={isAdmin} />
               <p className="text-blue-500 font-bold tracking-[0.4em] text-xs mb-12 uppercase">Candidate for Nursing School</p>
               <ImageUploader src={portData.profile.mainPhoto} className="w-72 h-[420px] rounded-[3rem] shadow-2xl mb-12 border-[10px] border-white" isAdmin={isAdmin} type="profile" onImageChange={(b) => updateField('profile.mainPhoto', b)} />
               <div className="bg-slate-50 w-full max-w-md p-6 rounded-[2.5rem] border border-slate-200 text-left">
                  <div className="flex justify-between items-end">
                    <div>
                       <span className="text-[10px] font-black text-blue-500 uppercase tracking-widest mb-1 block">ชื่อโรงเรียน</span>
                       <EditableText value={portData.profile.school} onChange={(v) => updateField('profile.school', v)} className="text-xl font-black text-slate-800" isAdmin={isAdmin} />
                    </div>
                    <div className="text-right">
                       <span className="text-[10px] font-black text-blue-500 uppercase tracking-widest mb-1 block">GPAX</span>
                       <EditableText value={portData.profile.gpax} onChange={(v) => updateField('profile.gpax', v)} className="text-3xl font-black text-blue-600 italic" isAdmin={isAdmin} />
                    </div>
                  </div>
               </div>
            </div>
          )}

          {/* หน้า 1: ประวัติส่วนตัว */}
          {activePage === 1 && (
            <div className="h-full animate-in slide-in-from-right duration-500">
              <div className="flex items-center gap-4 mb-10 border-b-8 border-blue-950 pb-6">
                <div className="bg-blue-950 text-white p-4 rounded-[1.5rem] shadow-lg"><UserIcon size={40}/></div>
                <h2 className="text-6xl font-black text-blue-950 italic tracking-tighter uppercase">Profile</h2>
              </div>
              <div className="grid grid-cols-1 md:grid-cols-12 gap-10">
                <div className="md:col-span-5 flex flex-col gap-6">
                  <ImageUploader src={portData.profile.profilePhoto} className="w-full aspect-[4/5] rounded-[3rem] shadow-2xl border-4 border-slate-100" isAdmin={isAdmin} type="profile" onImageChange={(b) => updateField('profile.profilePhoto', b)} />
                  <div className="bg-blue-950 p-8 rounded-[2.5rem] text-white shadow-xl relative overflow-hidden">
                    <Heart className="absolute -right-4 -bottom-4 text-white/5" size={100}/>
                    <h3 className="text-blue-400 font-black text-[10px] uppercase tracking-[0.2em] mb-4 border-b border-white/10 pb-2 flex items-center gap-2"><Heart size={14}/> คติประจำใจ</h3>
                    <EditableText multiline value={portData.profile.motto} onChange={(v) => updateField('profile.motto', v)} className="bg-transparent text-white font-black italic text-xl leading-relaxed" isAdmin={isAdmin} />
                  </div>
                </div>
                <div className="md:col-span-7 space-y-8">
                  <div className="bg-slate-50 p-8 rounded-[3rem] shadow-inner border border-slate-200 grid grid-cols-2 gap-6">
                    <div className="col-span-2">
                       <span className="text-[10px] font-black text-blue-600 uppercase tracking-widest">ชื่อ-นามสกุล</span>
                       <EditableText value={portData.profile.name} onChange={(v) => updateField('profile.name', v)} className="text-2xl font-black text-slate-800 block mt-1" isAdmin={isAdmin} />
                    </div>
                    <div>
                       <span className="text-[10px] font-black text-blue-600 uppercase tracking-widest">ชื่อเล่น</span>
                       <EditableText value={portData.profile.nickname} onChange={(v) => updateField('profile.nickname', v)} className="text-xl font-black text-slate-700 block mt-1" isAdmin={isAdmin} />
                    </div>
                    <div>
                       <span className="text-[10px] font-black text-blue-600 uppercase tracking-widest">หมู่เลือด</span>
                       <EditableText value={portData.profile.bloodType} onChange={(v) => updateField('profile.bloodType', v)} className="text-xl font-black text-red-600 block mt-1 uppercase" isAdmin={isAdmin} />
                    </div>
                    <div className="col-span-2">
                       <span className="text-[10px] font-black text-blue-600 uppercase tracking-widest">ที่อยู่</span>
                       <EditableText multiline value={portData.profile.address} onChange={(v) => updateField('profile.address', v)} className="text-sm font-bold text-slate-500 block mt-1 leading-relaxed" isAdmin={isAdmin} />
                    </div>
                  </div>
                  <div className="bg-blue-50 p-8 rounded-[3rem] border-2 border-blue-100 space-y-4 shadow-sm">
                    <h3 className="text-blue-700 font-black text-[10px] uppercase tracking-widest flex items-center gap-2 mb-4"><Phone size={14}/> ช่องทางการติดต่อ</h3>
                    <div className="grid grid-cols-1 gap-3">
                      <div className="flex items-center gap-4 bg-white p-4 rounded-2xl shadow-sm"><Phone className="text-blue-500" size={18}/><EditableText value={portData.profile.contact.tel} onChange={(v) => updateField('profile.contact.tel', v)} className="font-black text-slate-700 flex-1" isAdmin={isAdmin} /></div>
                      <div className="flex items-center gap-4 bg-white p-4 rounded-2xl shadow-sm"><Mail className="text-blue-500" size={18}/><EditableText value={portData.profile.contact.email} onChange={(v) => updateField('profile.contact.email', v)} className="font-black text-slate-700 flex-1" isAdmin={isAdmin} /></div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          )}

          {/* หน้า 2: ประวัติการศึกษา */}
          {activePage === 2 && (
            <div className="h-full">
              <h2 className="text-6xl font-black text-blue-950 border-b-8 border-blue-950 pb-6 mb-12 uppercase italic tracking-tighter">Education</h2>
              <div className="space-y-6">
                {portData.profile.education.map((edu, idx) => (
                  <div key={idx} className="flex items-center gap-8 bg-slate-50 p-8 rounded-[3rem] border-2 border-slate-100 shadow-sm transition-transform hover:scale-[1.02]">
                    <div className="bg-blue-600 p-6 rounded-[2rem] text-white shadow-xl"><GraduationCap size={32} /></div>
                    <div className="flex-1 grid grid-cols-1 md:grid-cols-3 gap-6">
                        <div><label className="text-[10px] font-black text-blue-500 uppercase tracking-widest mb-1 block">ระดับชั้น</label><EditableText value={edu.level} onChange={(v) => { let e = [...portData.profile.education]; e[idx].level = v; updateField('profile.education', e); }} className="text-xl font-black text-slate-800" isAdmin={isAdmin} /></div>
                        <div><label className="text-[10px] font-black text-blue-500 uppercase tracking-widest mb-1 block">สถานศึกษา</label><EditableText value={edu.school} onChange={(v) => { let e = [...portData.profile.education]; e[idx].school = v; updateField('profile.education', e); }} className="font-bold text-slate-600" isAdmin={isAdmin} /></div>
                        <div className="md:text-right"><label className="text-[10px] font-black text-blue-500 uppercase tracking-widest mb-1 block">GPAX</label><EditableText value={edu.gpax} onChange={(v) => { let e = [...portData.profile.education]; e[idx].gpax = v; updateField('profile.education', e); }} className="text-3xl font-black text-blue-600 italic" isAdmin={isAdmin} /></div>
                    </div>
                  </div>
                ))}
              </div>
            </div>
          )}

          {/* หน้า 3: SOP */}
          {activePage === 3 && (
            <div className="h-full flex flex-col">
              <h2 className="text-6xl font-black text-blue-950 border-b-8 border-blue-950 pb-6 mb-12 uppercase italic tracking-tighter">SOP</h2>
              <div className="bg-slate-50 p-12 rounded-[4rem] flex-1 border-4 border-slate-100 shadow-inner relative flex flex-col">
                <div className="flex items-center gap-3 mb-8 text-blue-600"><BookOpen size={30} /><span className="text-2xl font-black italic">เหตุผลที่อยากเรียนพยาบาล</span></div>
                <EditableText multiline value={portData.profile.sop_content} onChange={(v) => updateField('profile.sop_content', v)} className="bg-transparent font-bold text-slate-700 leading-[2.2] text-xl italic text-justify flex-1 block w-full" isAdmin={isAdmin} />
              </div>
            </div>
          )}

          {/* หน้า 4-6: กิจกรรม */}
          {(activePage >= 4 && activePage <= 6) && (
            <div className="h-full">
               <h2 className="text-6xl font-black text-blue-950 border-b-8 border-blue-950 pb-6 mb-12 uppercase italic tracking-tighter">
                 {activePage === 4 ? 'Academic' : activePage === 5 ? 'Volunteer' : 'Leadership'}
               </h2>
               <div className="space-y-12">
                 {(portData.profile[activePage === 4 ? 'academic_activities' : activePage === 5 ? 'volunteer_activities' : 'leadership_activities'] || []).map((act, i) => (
                    <div key={act.id} className="flex flex-col md:flex-row gap-10 items-center bg-slate-50 p-8 rounded-[3.5rem] border border-slate-200 shadow-sm transition-all hover:shadow-xl">
                      <ImageUploader 
                        src={act.imageUrl} 
                        className="w-full md:w-1/2 aspect-[4/3] rounded-[2.5rem] shrink-0 shadow-2xl border-8 border-white" 
                        isAdmin={isAdmin} 
                        type={activePage === 4 ? 'nurse_activity' : activePage === 5 ? 'volunteer' : 'leadership'} 
                        onImageChange={(b) => updateItem(activePage === 4 ? 'academic_activities' : activePage === 5 ? 'volunteer_activities' : 'leadership_activities', act.id, 'imageUrl', b)} 
                      />
                      <div className="flex-1">
                        <EditableText value={act.title} onChange={(v) => updateItem(activePage === 4 ? 'academic_activities' : activePage === 5 ? 'volunteer_activities' : 'leadership_activities', act.id, 'title', v)} className="text-3xl font-black text-blue-900 block mb-4 uppercase leading-tight" isAdmin={isAdmin} />
                        <EditableText multiline value={act.desc} onChange={(v) => updateItem(activePage === 4 ? 'academic_activities' : activePage === 5 ? 'volunteer_activities' : 'leadership_activities', act.id, 'desc', v)} className="text-lg font-bold text-slate-500 leading-relaxed block w-full" isAdmin={isAdmin} />
                      </div>
                    </div>
                 ))}
               </div>
            </div>
          )}

          {/* หน้า 7-9: เกียรติบัตร */}
          {activePage >= 7 && (
            <div className="h-full">
               <h2 className="text-6xl font-black text-blue-950 border-b-8 border-blue-950 pb-6 mb-12 uppercase italic tracking-tighter">Certificates</h2>
               <div className="grid grid-cols-1 md:grid-cols-2 gap-8">
                 {(portData.profile[activePage === 7 ? 'certificates_high' : activePage === 8 ? 'certificates_moral' : 'certificates_others'] || []).map(cert => (
                    <div key={cert.id} className="bg-white rounded-[3rem] shadow-2xl border-2 border-slate-100 flex flex-col overflow-hidden transition-transform hover:-translate-y-2">
                       <ImageUploader src={cert.imageUrl} className="aspect-[4/3]" isAdmin={isAdmin} type="certificate" onImageChange={(b) => updateItem(activePage === 7 ? 'certificates_high' : activePage === 8 ? 'certificates_moral' : 'certificates_others', cert.id, 'imageUrl', b)} />
                       <div className="p-8 bg-slate-50 flex-1">
                          <EditableText value={cert.title} onChange={(v) => updateItem(activePage === 7 ? 'certificates_high' : activePage === 8 ? 'certificates_moral' : 'certificates_others', cert.id, 'title', v)} className="text-xl font-black text-blue-950 block mb-2 leading-tight" isAdmin={isAdmin} />
                          <div className="flex items-center gap-2 text-slate-400 font-black uppercase text-[10px] tracking-widest mt-4">
                             <Award size={16} className="text-blue-500"/>
                             <EditableText value={cert.issuer} onChange={(v) => updateItem(activePage === 7 ? 'certificates_high' : activePage === 8 ? 'certificates_moral' : 'certificates_others', cert.id, 'issuer', v)} className="bg-transparent" isAdmin={isAdmin} />
                          </div>
                       </div>
                    </div>
                 ))}
               </div>
            </div>
          )}

          <div className="mt-auto pt-10 flex justify-between items-center border-t-2 border-slate-100">
             <button disabled={activePage === 0} onClick={() => {setActivePage(p => p - 1); window.scrollTo(0,0);}} className="text-slate-400 font-black uppercase text-[10px] tracking-widest flex items-center gap-2 hover:text-blue-600 disabled:opacity-0 transition-all">
                <ChevronLeft size={20}/> ก่อนหน้า
             </button>
             <div className="flex gap-2">
               {pages.map((_, i) => (
                 <div key={i} className={`h-2 rounded-full transition-all duration-300 ${activePage === i ? 'bg-blue-600 w-10 shadow-lg shadow-blue-200' : 'bg-slate-200 w-2'}`}></div>
               ))}
             </div>
             <button disabled={activePage === 9} onClick={() => {setActivePage(p => p + 1); window.scrollTo(0,0);}} className="text-blue-950 font-black uppercase text-[10px] tracking-widest flex items-center gap-2 hover:text-blue-600 transition-all">
                ถัดไป <ChevronRight size={20}/>
             </button>
          </div>

          {showLogin && (
            <div className="fixed inset-0 bg-blue-950/90 backdrop-blur-xl z-[200] flex items-center justify-center p-4">
              <div className="bg-white p-12 rounded-[4rem] shadow-2xl w-full max-w-sm text-center border-4 border-blue-600">
                <NurseIcon size={48} className="mx-auto text-blue-600 mb-6 animate-pulse"/>
                <h2 className="font-black text-3xl text-blue-950 mb-2 uppercase italic tracking-tighter">สิทธิ์ผู้ดูแล</h2>
                <p className="text-slate-400 font-bold text-xs uppercase tracking-widest mb-8">กรุณากรอกรหัสผ่าน</p>
                <input 
                  type="password" value={password} onChange={(e) => setPassword(e.target.value)} 
                  placeholder="••••" className="w-full p-5 border-4 border-slate-100 rounded-[2rem] mb-6 text-center text-4xl outline-none focus:border-blue-500 transition-all"
                  onKeyPress={(e) => e.key === 'Enter' && (password === '1234' ? (setIsAdmin(true), setShowLogin(false), setPassword('')) : alert("รหัสผ่านไม่ถูกต้อง"))}
                  autoFocus
                />
                <button onClick={() => { if(password==='1234') {setIsAdmin(true); setShowLogin(false); setPassword('');} else alert("รหัสผ่านไม่ถูกต้อง"); }} className="w-full py-5 bg-blue-600 text-white rounded-[2rem] font-black text-xl shadow-xl hover:bg-blue-500 transition-all active:scale-95 uppercase italic tracking-tighter">เข้าสู่ระบบ</button>
                <button onClick={() => setShowLogin(false)} className="mt-6 text-slate-300 font-black text-[10px] uppercase tracking-widest">ยกเลิก</button>
              </div>
            </div>
          )}
        </main>
      </div>
      <footer className="bg-white border-t border-slate-200 p-6 text-center">
         <p className="text-[10px] font-black text-slate-400 uppercase tracking-[0.2em]">Designed for TCAS Nursing Excellence &copy; 2026</p>
      </footer>
    </div>
  );
};

export default App;
