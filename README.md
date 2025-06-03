import React, { useState, useEffect } from 'react';
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";
import { BarChart, Bar, XAxis, YAxis, Tooltip, ResponsiveContainer, Legend } from 'recharts';
import { toast } from 'react-hot-toast';

export default function RevenueGrowthCalculator() {
  const [formData, setFormData] = useState({
    currentRevenue: '',
    targetRevenue: '',
    currentClients: '',
    closeRate: '',
    salesCycle: '',
    meetingsPerMonth: '',
    churnRate: '',
    priceIncreases: '10,25,50',
    meetingIncreases: '50,100,200',
  });

  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    const savedData = localStorage.getItem('revenueGrowthForm');
    if (savedData) {
      setFormData(JSON.parse(savedData));
    }
  }, []);

  useEffect(() => {
    localStorage.setItem('revenueGrowthForm', JSON.stringify(formData));
  }, [formData]);

  const handleChange = (e) => {
    setFormData({ ...formData, [e.target.name]: e.target.value });
  };

  const resetForm = () => {
    const defaultData = {
      currentRevenue: '',
      targetRevenue: '',
      currentClients: '',
      closeRate: '',
      salesCycle: '',
      meetingsPerMonth: '',
      churnRate: '',
      priceIncreases: '10,25,50',
      meetingIncreases: '50,100,200',
    };
    setFormData(defaultData);
    setResults([]);
    localStorage.removeItem('revenueGrowthForm');
    toast.success('Form reset successfully!');
  };

  const calculateGrowth = async () => {
    setLoading(true);
    try {
      const res = await fetch('/api/calculate-growth', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData),
      });

      if (!res.ok) throw new Error('Failed to calculate');

      const data = await res.json();
      setResults(data.scenarios);
      toast.success('Calculation complete!');
    } catch (error) {
      toast.error('Something went wrong while calculating.');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="max-w-4xl mx-auto p-4 space-y-4">
      <h1 className="text-2xl font-bold">Revenue Growth Calculator</h1>
      <p className="text-sm text-gray-500">Answer the 7 prompts below and test multiple pricing & meeting volume scenarios.</p>

      <Input name="currentRevenue" placeholder="1. Current annual revenue ($)" value={formData.currentRevenue} onChange={handleChange} />
      <Input name="targetRevenue" placeholder="2. Target annual revenue ($)" value={formData.targetRevenue} onChange={handleChange} />
      <Input name="currentClients" placeholder="3. Current paying clients" value={formData.currentClients} onChange={handleChange} />
      <Input name="closeRate" placeholder="4. Sales conversion rate (e.g. 0.2 for 20%)" value={formData.closeRate} onChange={handleChange} />
      <Input name="salesCycle" placeholder="5. Average sales cycle (months)" value={formData.salesCycle} onChange={handleChange} />
      <Input name="meetingsPerMonth" placeholder="6. Sales meetings per month" value={formData.meetingsPerMonth} onChange={handleChange} />
      <Input name="churnRate" placeholder="7. Monthly churn rate (e.g. 0.05 for 5%)" value={formData.churnRate} onChange={handleChange} />
      <Input name="priceIncreases" placeholder="Price Increase % (comma separated, e.g. 10,25,50)" value={formData.priceIncreases} onChange={handleChange} />
      <Input name="meetingIncreases" placeholder="Meeting Volume Increase % (comma separated, e.g. 50,100,200)" value={formData.meetingIncreases} onChange={handleChange} />

      <div className="flex space-x-2">
        <Button onClick={calculateGrowth} disabled={loading}>
          {loading ? (
            <>
              <svg className="animate-spin h-5 w-5 mr-2 text-white" viewBox="0 0 24 24">
                <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4" />
                <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v8z" />
              </svg>
              Calculating...
            </>
          ) : 'Calculate'}
        </Button>
        <Button variant="outline" onClick={resetForm}>Reset</Button>
      </div>

      {results.length > 0 && (
        <div className="space-y-6 mt-6">
          <div className="h-96">
            <ResponsiveContainer width="100%" height="100%">
              <BarChart data={results} margin={{ top: 20, right: 30, left: 20, bottom: 5 }}>
                <XAxis dataKey="scenarioLabel" tick={{ fontSize: 10 }} angle={-45} textAnchor="end" interval={0} height={80} />
                <YAxis />
                <Tooltip />
                <Legend />
                <Bar dataKey="monthsToTarget" fill="#3b82f6" name="Months to Target" />
              </BarChart>
            </ResponsiveContainer>
          </div>

          {results.map((r, index) => (
            <Card key={index}>
              <CardContent className="space-y-2">
                <p><strong>Scenario:</strong> +{r.priceIncrease}% price, +{r.meetingIncrease}% meetings</p>
                <p><strong>New Avg Revenue per Client:</strong> ${r.newAvgRevenuePerClient.toFixed(2)}</p>
                <p><strong>Additional Clients Needed:</strong> {r.additionalClientsNeeded}</p>
                <p><strong>Sales Calls Needed:</strong> {r.callsNeeded}</p>
                <p><strong>Adjusted Monthly Meetings:</strong> {r.adjustedMeetings.toFixed(1)}</p>
                <p><strong>Estimated Months to Target:</strong> {r.monthsToTarget}</p>
              </CardContent>
            </Card>
          ))}
        </div>
      )}
    </div>
  );
}
