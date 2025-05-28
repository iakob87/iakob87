// patient_record_app: React + Tailwind PWA with patient search and calendar alerts + localStorage backup

import React, { useState, useEffect } from 'react';
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Calendar } from "@/components/ui/calendar";

const initialPatient = {
  name: '',
  age: '',
  gender: '',
  phone: '',
  visits: []
};

const initialVisit = {
  date: '',
  symptoms: '',
  diagnosis: '',
  visualAcuity: '',
  fundusFindings: '',
  octResults: '',
  iopReadings: '',
  notes: '',
  followUpDate: '',
  image: null
};

const diagnosisOptions = [
  { label: "Cataract", code: "H25.1" },
  { label: "Glaucoma - Primary Open-Angle", code: "H40.11" },
  { label: "Glaucoma - Angle Closure", code: "H40.21" },
  { label: "Diabetic Retinopathy", code: "E11.319" },
  { label: "Age-related Macular Degeneration", code: "H35.31" },
  { label: "Retinal Detachment", code: "H33.0" },
  { label: "Uveitis", code: "H20.9" },
  { label: "Normal", code: "Z01.00" },
  { label: "Other", code: "" }
];

export default function PatientRecordApp() {
  const [patients, setPatients] = useState([]);
  const [patient, setPatient] = useState(initialPatient);
  const [visit, setVisit] = useState(initialVisit);
  const [selectedImage, setSelectedImage] = useState(null);
  const [calendarEvents, setCalendarEvents] = useState([]);
  const [searchTerm, setSearchTerm] = useState('');

  useEffect(() => {
    const storedPatients = localStorage.getItem('patients');
    const storedCalendar = localStorage.getItem('calendarEvents');
    if (storedPatients) setPatients(JSON.parse(storedPatients));
    if (storedCalendar) setCalendarEvents(JSON.parse(storedCalendar));
  }, []);

  useEffect(() => {
    const checkReminders = () => {
      const today = new Date();
      patients.forEach(p => {
        p.visits.forEach(v => {
          const followUp = new Date(v.followUpDate);
          const diff = (followUp - today) / (1000 * 60 * 60 * 24);
          if (diff <= 2 && diff >= 0) {
            alert(`Reminder: Call ${p.name} for follow-up on ${v.followUpDate}`);
          }
        });
      });
    };
    checkReminders();
  }, [patients]);

  const handleImageUpload = (e) => {
    const file = e.target.files[0];
    if (file) setSelectedImage(URL.createObjectURL(file));
  };

  const handleAddVisit = () => {
    const updatedPatient = {
      ...patient,
      visits: [...patient.visits, { ...visit, image: selectedImage }]
    };
    const updatedPatients = [...patients, updatedPatient];
    const updatedCalendar = [...calendarEvents, { title: patient.name, date: visit.followUpDate }];

    setPatients(updatedPatients);
    setCalendarEvents(updatedCalendar);
    localStorage.setItem('patients', JSON.stringify(updatedPatients));
    localStorage.setItem('calendarEvents', JSON.stringify(updatedCalendar));

    setPatient(initialPatient);
    setVisit(initialVisit);
    setSelectedImage(null);
  };

  const filteredPatients = patients.filter(p => p.name.toLowerCase().includes(searchTerm.toLowerCase()));

  return (
    <div className="p-4 grid gap-4 max-w-2xl mx-auto">
      <Card>
        <CardContent className="space-y-2">
          <h2 className="text-xl font-semibold">Add Patient</h2>
          <Input placeholder="Full Name" value={patient.name} onChange={e => setPatient({ ...patient, name: e.target.value })} />
          <Input placeholder="Age" value={patient.age} onChange={e => setPatient({ ...patient, age: e.target.value })} />
          <select className="input w-full p-2 border rounded" value={patient.gender} onChange={e => setPatient({ ...patient, gender: e.target.value })}>
            <option value="">Select Gender</option>
            <option value="Male">Male</option>
            <option value="Female">Female</option>
            <option value="Other">Other</option>
          </select>
          <Input placeholder="Phone Number" value={patient.phone} onChange={e => setPatient({ ...patient, phone: e.target.value })} />
        </CardContent>
      </Card>

      <Card>
        <CardContent className="space-y-2">
          <h2 className="text-xl font-semibold">Add Observation</h2>
          <Input type="date" value={visit.date} onChange={e => setVisit({ ...visit, date: e.target.value })} />
          <select className="input w-full p-2 border rounded" value={visit.symptoms} onChange={e => setVisit({ ...visit, symptoms: e.target.value })}>
            <option value="">Select Symptom</option>
            <option value="Blurred Vision">Blurred Vision</option>
            <option value="Floaters">Floaters</option>
            <option value="Sudden Vision Loss">Sudden Vision Loss</option>
            <option value="Pain">Pain</option>
            <option value="Redness">Redness</option>
          </select>
          <select className="input w-full p-2 border rounded" value={visit.diagnosis} onChange={e => setVisit({ ...visit, diagnosis: e.target.value })}>
            <option value="">Select Diagnosis</option>
            {diagnosisOptions.map((d, i) => (
              <option key={i} value={d.label}>{d.label}{d.code ? ` (${d.code})` : ''}</option>
            ))}
          </select>
          <Input placeholder="Visual Acuity" value={visit.visualAcuity} onChange={e => setVisit({ ...visit, visualAcuity: e.target.value })} />
          <Textarea placeholder="Fundus Findings" value={visit.fundusFindings} onChange={e => setVisit({ ...visit, fundusFindings: e.target.value })} />
          <Textarea placeholder="OCT Results" value={visit.octResults} onChange={e => setVisit({ ...visit, octResults: e.target.value })} />
          <Input placeholder="IOP Readings" value={visit.iopReadings} onChange={e => setVisit({ ...visit, iopReadings: e.target.value })} />
          <Textarea placeholder="Free-form Notes" value={visit.notes} onChange={e => setVisit({ ...visit, notes: e.target.value })} />
          <Input type="date" value={visit.followUpDate} onChange={e => setVisit({ ...visit, followUpDate: e.target.value })} />
          <Input type="file" accept="image/*" capture="environment" onChange={handleImageUpload} />
          {selectedImage && <img src={selectedImage} alt="Uploaded" className="w-40 rounded" />}
          <Button onClick={handleAddVisit}>Save Patient & Visit</Button>
        </CardContent>
      </Card>

      <Card>
        <CardContent>
          <h2 className="text-xl font-semibold mb-2">Search Patients</h2>
          <Input placeholder="Search by name..." value={searchTerm} onChange={e => setSearchTerm(e.target.value)} />
          <ul className="mt-2 list-disc pl-5">
            {filteredPatients.map((p, i) => (
              <li key={i} className="mb-1">
                {p.name} - {p.phone} ({p.gender})
              </li>
            ))}
          </ul>
        </CardContent>
      </Card>

      <Card>
        <CardContent>
          <h2 className="text-xl font-semibold mb-2">Upcoming Follow-Ups</h2>
          <ul className="list-disc pl-5">
            {calendarEvents.map((event, index) => (
              <li key={index}>{event.date}: {event.title}</li>
            ))}
          </ul>
        </CardContent>
      </Card>
    </div>
  );
}
- 👋 Hi, I’m @iakob87
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
iakob87/iakob87 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
