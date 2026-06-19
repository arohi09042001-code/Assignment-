# Assignment-
 Build a responsive web application using React/Next.js, TypeScript, AWS and OpenAI API to monitor AWS cloud spending and provide AI-powered budget insights
To implement the Budget Overview metrics component elegantly in your React/Next.js dashboard, you should present these high-level KPIs right at the top of the page. This gives users an instant snapshot of their financial health before they dive into the graphs and AI insights.

Here is a ready-to-use React component structured with TypeScript, Tailwind CSS, and Shadcn UI card components.

🎨 Budget Overview Component (BudgetCards.tsx)
This component takes the raw financial numbers, calculates the utilization percentage automatically, and dynamically changes colors (e.g., turning red if utilization exceeds 90%)
import React from 'react';
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { DollarSign, Percent, Wallet, ArrowUpRight } from "lucide-react";

interface BudgetOverviewProps {
  monthlyBudget: number;
  currentSpend: number;
}

export default function BudgetCards({ monthlyBudget, currentSpend }: BudgetOverviewProps) {
  const remainingBudget = monthlyBudget - currentSpend;
  const utilizationRate = monthlyBudget > 0 ? (currentSpend / monthlyBudget) * 100 : 0;
  
  // Dynamic color state based on budget risk
  const isOverBudget = currentSpend > monthlyBudget;
  const isHighUtilization = utilizationRate >= 90;

  return (
    <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-4">
      {/* Monthly Budget Card */}
      <Card>
        <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
          <CardTitle className="text-sm font-medium text-muted-foreground">Monthly Budget</CardTitle>
          <DollarSign className="h-4 w-4 text-muted-foreground" />
        </CardHeader>
        <CardContent>
          <div className="text-2xl font-bold">${monthlyBudget.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 })}</div>
          <p className="text-xs text-muted-foreground">Allocated cloud cap</p>
        </CardContent>
      </Card>

      {/* Current Spend Card */}
      <Card>
        <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
          <CardTitle className="text-sm font-medium text-muted-foreground">Current Spend</CardTitle>
          <ArrowUpRight className={`h-4 w-4 ${isOverBudget ? 'text-destructive' : 'text-muted-foreground'}`} />
        </CardHeader>
        <CardContent>
          <div className={`text-2xl font-bold ${isOverBudget ? 'text-destructive' : ''}`}>
            ${currentSpend.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 })}
          </div>
          <p className="text-xs text-muted-foreground">Month-to-date AWS pull</p>
        </CardContent>
      </Card>

      {/* Remaining Budget Card */}
      <Card>
        <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
          <CardTitle className="text-sm font-medium text-muted-foreground">Remaining Budget</CardTitle>
          <Wallet className={`h-4 w-4 ${remainingBudget < 0 ? 'text-destructive' : 'text-muted-foreground'}`} />
        </CardHeader>
        <CardContent>
          <div className={`text-2xl font-bold ${remainingBudget < 0 ? 'text-destructive' : 'text-green-600 dark:text-green-400'}`}>
            ${remainingBudget.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 })}
          </div>
          <p className="text-xs text-muted-foreground">
            {remainingBudget < 0 ? 'Over allocated limit' : 'Available to spend'}
          </p>
        </CardContent>
      </Card>

      {/* Utilization Card */}
      <Card>
        <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
          <CardTitle className="text-sm font-medium text-muted-foreground">Utilization</CardTitle>
          <Percent className="h-4 w-4 text-muted-foreground" />
        </CardHeader>
        <CardContent>
          <div className={`text-2xl font-bold ${isHighUtilization ? 'text-destructive font-extrabold' : 'text-primary'}`}>
            {utilizationRate.toFixed(1)}%
          </div>
          {/* Simple CSS Progress Bar */}
          <div className="mt-2 h-2 w-full rounded-full bg-secondary overflow-hidden">
            <div 
              className={`h-full rounded-full transition-all duration-500 ${
                isHighUtilization ? 'bg-destructive' : 'bg-primary'
              }`}
              style={{ width: `${Math.min(utilizationRate, 100)}%` }}
            />
          </div>
        </CardContent>
      </Card>
    </div>
  );
} 
🛠️ Data Architecture IntegrationTo feed data into this component securely from AWS Cost Explorer on your backend, you can write an internal utility function.Mathematically, the calculations happening under the hood are simple but vital for your UI state:Remaining Budget: $\text{Budget} - \text{Spend}$Utilization Rate: $\left( \frac{\text{Spend}}{\text{Budget}} \right) \times 100$Here is how you would call it inside your dashboard's server component (src/app/page.tsx):
import BudgetCards from "@/components/BudgetCards";
// Imagine this helper queries the @aws-sdk/client-cost-explorer
import { getMonthToDateSpend } from "@/lib/aws-actions"; 

export default async function DashboardPage() {
  // Fetch real data from AWS, fallback to mock if API fails/unlinked
  const currentSpend = await getMonthToDateSpend() || 1420.50; 
  const monthlyBudget = 2000.00; // This can come from a local DB or user config

  return (
    <main className="p-6 space-y-6">
      <h1 className="text-3xl font-bold tracking-tight">AWS Cost Monitor</h1>
      
      {/* Render the core budget metrics */}
      <BudgetCards monthlyBudget={monthlyBudget} currentSpend={currentSpend} />
      
      {/* Graphs and AI Insights sections would go down here */}
    </main>
  );
} 
To make your Cost Analytics dashboard truly interactive and professional, you can use Recharts, which is the gold standard for React charts due to its native responsive containers and easy tailwind styling.

Here is the implementation code, data structures, and strategies for your daily, weekly, monthly, and service-wise views.

📊 The Cost Analytics Component (CostCharts.tsx)
This component leverages a single, clean layout with an elegant tab-switcher (using Shadcn UI Tabs or native state) so the user can fluidly jump between the different time horizons and granularities
"use client";

import React, { useState } from 'react';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from "@/components/ui/card";
import { Tabs, TabsList, TabsTrigger } from "@/components/ui/tabs";
import { BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer, LineChart, Line, Legend } from 'recharts';

// --- MOCK DATA STRUCTURES MATCHING AWS COST EXPLORER OUTPUTS ---
const dailyData = [
  { date: '06/12', Spend: 45.20 },
  { date: '06/13', Spend: 48.90 },
  { date: '06/14', Spend: 52.10 },
  { date: '06/15', Spend: 78.40 }, // Spike! Great for AI demo
  { date: '06/16', Spend: 50.10 },
  { date: '06/17', Spend: 47.30 },
  { date: '06/18', Spend: 49.00 },
];

const weeklyData = [
  { week: 'Wk 21', Spend: 320.50 },
  { week: 'Wk 22', Spend: 345.00 },
  { week: 'Wk 23', Spend: 410.20 },
  { week: 'Wk 24', Spend: 385.10 },
];

const monthlyData = [
  { month: 'Jan', Spend: 1200 },
  { month: 'Feb', Spend: 1350 },
  { month: 'Mar', Spend: 1310 },
  { month: 'Apr', Spend: 1420 },
  { month: 'May', Spend: 1550 },
  { month: 'Jun', Spend: 1420.50 },
];

const serviceData = [
  { service: 'Amazon EC2', Spend: 620.40, fill: '#FF9900' }, // AWS Orange
  { service: 'Amazon RDS', Spend: 340.10, fill: '#3F7E44' },  // Database Green
  { service: 'Amazon S3', Spend: 180.50, fill: '#116H22' },   // Storage Blue
  { service: 'AWS Lambda', Spend: 95.20, fill: '#E05243' },   // Serverless Red
  { service: 'Others', Spend: 184.30, fill: '#7AA116' },
];

export default function CostAnalytics() {
  const [activeTab, setActiveTab] = useState('daily');

  // Format currency inside tooltips
  const formatCurrency = (value: number) => `$${value.toLocaleString('en-US', { minimumFractionDigits: 2 })}`;

  return (
    <Card className="w-full col-span-4">
      <CardHeader className="flex flex-col sm:flex-row items-start sm:items-center justify-between space-y-2 sm:space-y-0 pb-7">
        <div>
          <CardTitle className="text-xl font-bold">Cost Analytics</CardTitle>
          <CardDescription>Visual breakdown of historical AWS cloud expenditure</CardDescription>
        </div>
        
        {/* Navigation Tabs */}
        <Tabs value={activeTab} onValueChange={setActiveTab} className="w-auto">
          <TabsList>
            <TabsTrigger value="daily">Daily</TabsTrigger>
            <TabsTrigger value="weekly">Weekly</TabsTrigger>
            <TabsTrigger value="monthly">Monthly</TabsTrigger>
            <TabsTrigger value="service">By Service</TabsTrigger>
          </TabsList>
        </Tabs>
      </CardHeader>

      <CardContent className="h-[350px] w-full">
        <ResponsiveContainer width="100%" height="100%">
          
          {/* DAILY VIEW: Line chart to show trends and sudden daily spikes */}
          {activeTab === 'daily' && (
            <LineChart data={dailyData} margin={{ top: 5, right: 10, left: -10, bottom: 0 }}>
              <CartesianGrid strokeDasharray="3 3" className="stroke-muted" />
              <XAxis dataKey="date" className="text-xs fill-muted-foreground" />
              <YAxis tickFormatter={(v) => `$${v}`} className="text-xs fill-muted-foreground" />
              <Tooltip formatter={formatCurrency} contentStyle={{ background: 'hsl(var(--background))', borderRadius: '8px' }} />
              <Line type="monotone" dataKey="Spend" stroke="hsl(var(--primary))" strokeWidth={2} activeDot={{ r: 6 }} />
            </LineChart>
          )}

          {/* WEEKLY VIEW: Clean Bar Chart */}
          {activeTab === 'weekly' && (
            <BarChart data={weeklyData} margin={{ top: 5, right: 10, left: -10, bottom: 0 }}>
              <CartesianGrid strokeDasharray="3 3" className="stroke-muted" />
              <XAxis dataKey="week" className="text-xs fill-muted-foreground" />
              <YAxis tickFormatter={(v) => `$${v}`} className="text-xs fill-muted-foreground" />
              <Tooltip formatter={formatCurrency} contentStyle={{ background: 'hsl(var(--background))', borderRadius: '8px' }} />
              <Bar dataKey="Spend" fill="hsl(var(--primary))" radius={[4, 4, 0, 0]} />
            </BarChart>
          )}

          {/* MONTHLY VIEW: Bar Chart for macro budget analysis */}
          {activeTab === 'monthly' && (
            <BarChart data={monthlyData} margin={{ top: 5, right: 10, left: -10, bottom: 0 }}>
              <CartesianGrid strokeDasharray="3 3" className="stroke-muted" />
              <XAxis dataKey="month" className="text-xs fill-muted-foreground" />
              <YAxis tickFormatter={(v) => `$${v}`} className="text-xs fill-muted-foreground" />
              <Tooltip formatter={formatCurrency} contentStyle={{ background: 'hsl(var(--background))', borderRadius: '8px' }} />
              <Bar dataKey="Spend" fill="hsl(var(--primary))" radius={[4, 4, 0, 0]} />
            </BarChart>
          )}

          {/* SERVICE-WISE VIEW: Horizontal Bar Chart to easily read service titles */}
          {activeTab === 'service' && (
            <BarChart data={serviceData} layout="vertical" margin={{ top: 5, right: 10, left: 20, bottom: 0 }}>
              <CartesianGrid strokeDasharray="3 3" className="stroke-muted" horizontal={false} />
              <XAxis type="number" tickFormatter={(v) => `$${v}`} className="text-xs fill-muted-foreground" />
              <YAxis dataKey="service" type="category" className="text-xs fill-muted-foreground" width={90} />
              <Tooltip formatter={formatCurrency} contentStyle={{ background: 'hsl(var(--background))', borderRadius: '8px' }} />
              <Bar dataKey="Spend" radius={[0, 4, 4, 0]} />
            </BarChart>
          )}

        </ResponsiveContainer>
      </CardContent>
    </Card>
  );
} 
🛠️ Mapping to AWS Cost Explorer API Types
When writing your server side data parser (src/lib/aws.ts), you will parse the output of AWS GetCostAndUsageCommand arrays into the flat JSON fields Recharts expects.

Here is the quick mapping cheat sheet for how your backend should map real AWS payloads to the charts:

Time Horizons Mapping
Daily: Set TimePeriod to the last 7 days, and Granularity to DAILY. Use Keys[0] from Group or TimePeriod.Start as the chart date.

Monthly: Set TimePeriod to the last 6 months, and Granularity to MONTHLY.

Service-wise: Group by dimensions using:
GroupBy: [{ Type: 'DIMENSION', Key: 'SERVICE' }]
To complete your core metrics and cost charts, a robust Threshold & Over-Budget Alerts system is essential. This component notifies users when spending approaches or breaches pre-configured limits (e.g., 80% or 100% of budget), providing clear visual flags and AI-driven mitigation steps.

Here is the implementation strategy, UI component, and logic for threshold tracking.

🚨 Alerts & Thresholds Component (BudgetAlerts.tsx)
This component evaluates the user's spending data, checks it against predefined thresholds, and dynamically displays warning or critical banners using Shadcn UI Alerts and Lucide Icons.
"use client";

import React from 'react';
import { Alert, AlertDescription, AlertTitle } from "@/components/ui/alert";
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from "@/components/ui/card";
import { AlertTriangle, AlertCircle, CheckCircle2, ShieldAlert } from "lucide-react";

interface BudgetAlertsProps {
  monthlyBudget: number;
  currentSpend: number;
}

export default function BudgetAlerts({ monthlyBudget, currentSpend }: BudgetAlertsProps) {
  const utilizationRate = monthlyBudget > 0 ? (currentSpend / monthlyBudget) * 100 : 0;
  
  // Define thresholds
  const WARNING_THRESHOLD = 80;
  const CRITICAL_THRESHOLD = 100;

  // Track triggered alerts
  const alerts = [];

  if (utilizationRate >= CRITICAL_THRESHOLD) {
    alerts.push({
      id: 'critical',
      variant: 'destructive' as const,
      icon: <AlertCircle className="h-4 w-4" />,
      title: "Budget Breached (Critical)",
      description: `Your current spend ($${currentSpend.toFixed(2)}) has exceeded your monthly budget configuration of $${monthlyBudget.toFixed(2)} by ${(utilizationRate - 100).toFixed(1)}%.`,
    });
  } else if (utilizationRate >= WARNING_THRESHOLD) {
    alerts.push({
      id: 'warning',
      variant: 'default' as const, // Can style custom colors via Tailwind
      icon: <AlertTriangle className="h-4 w-4 text-amber-500" />,
      title: "Approaching Budget Threshold (Warning)",
      description: `You have utilized ${utilizationRate.toFixed(1)}% of your monthly budget. Current spend is $${currentSpend.toFixed(2)} out of $${monthlyBudget.toFixed(2)}.`,
      className: "border-amber-500 bg-amber-50 dark:bg-amber-950/20 text-amber-900 dark:text-amber-200",
    });
  }

  return (
    <Card className="w-full">
      <CardHeader>
        <CardTitle className="text-xl font-bold flex items-center gap-2">
          <ShieldAlert className="h-5 w-5 text-muted-foreground" />
          Budget Anomalies & Alerts
        </CardTitle>
        <CardDescription>Real-time status of your pre-configured spending limits</CardDescription>
      </CardHeader>
      <CardContent className="space-y-4">
        {alerts.length === 0 ? (
          <div className="flex items-center gap-3 p-4 border rounded-lg bg-green-50 dark:bg-green-950/20 border-green-500/30">
            <CheckCircle2 className="h-5 w-5 text-green-600 dark:text-green-400" />
            <div>
              <p className="text-sm font-medium text-green-900 dark:text-green-200">All costs normal</p>
              <p className="text-xs text-green-700 dark:text-green-400">Spending is well within safe thresholds (${utilizationRate.toFixed(1)}% used).</p>
            </div>
          </div>
        ) : (
          alerts.map((alert) => (
            <Alert key={alert.id} variant={alert.variant} className={alert.className}>
              {alert.icon}
              <AlertTitle className="font-semibold">{alert.title}</AlertTitle>
              <AlertDescription className="text-sm opacity-90">
                {alert.description}
              </AlertDescription>
            </Alert>
          ))
        )}
      </CardContent>
    </Card>
  );
} 
🤖 Bridging Alerts with OpenAI API Insights
When an alert status transitions from safe to Warning or Critical, you shouldn't just show an error message; you should use this state to dynamically alter the prompt sent to the OpenAI API (gpt-4o-mini) so it acts as an immediate incident response tool.

In your Server Action (src/actions/insights.ts), construct your system context conditional to the alert payload
export async function generateBudgetInsights(currentSpend: number, monthlyBudget: number, cloudMetrics: any) {
  const utilization = (currentSpend / monthlyBudget) * 100;
  
  let structuralPrompt = "You are an AWS Cloud Economist analyzing basic corporate infrastructure spend.";
  
  if (utilization >= 100) {
    structuralPrompt += ` CRITICAL EMERGENCY: The company has officially breached its budget cap! Focus exclusively on identifying rapid cost reduction steps, pointing out safe assets to immediately stop/terminate (e.g., idle dev EC2 instances, unattached EBS volumes), and write it in a structured, actionable layout.`;
  } else if (utilization >= 80) {
    structuralPrompt += ` ALERT: The customer has breached the 80% cautionary threshold. Provide preventative mitigation guidance to stall further unexpected spikes over the remaining days of the month.`;
  }

  // Pass structuralPrompt as System Message to openai.chat.completions.create()...
} 
To complement your dashboard's analytics and static alert banners, a Conversational AI Chatbot serves as the interactive core of your application. This allows users to ask ad-hoc questions about their AWS bill (e.g., "Why did my S3 cost double yesterday?" or "How can I cut costs on my RDS instances?") and receive contextual, data-backed answers from OpenAI.

Here is the implementation guide for an interactive chatbot using Next.js Server Actions, TypeScript, Tailwind CSS, and Shadcn UI.

💬 The Chatbot Component (CostChatbot.tsx)
This component features an auto-scrolling message history window and a loading state that simulates streaming or processing while waiting for the server backend to respond 
"use client";

import React, { useState, useRef, useEffect } from 'react';
import { Card, CardContent, CardFooter, CardHeader, CardTitle, CardDescription } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { ScrollArea } from "@/components/ui/scroll-area";
import { Bot, User, Send, Loader2, Sparkles } from "lucide-react";
import { askFinOpsBot } from "@/actions/chat"; // Server action for OpenAI handling

interface Message {
  id: string;
  role: 'user' | 'assistant';
  content: string;
  timestamp: Date;
}

export default function CostChatbot() {
  const [messages, setMessages] = useState<Message[]>([
    {
      id: 'welcome',
      role: 'assistant',
      content: "👋 Hello! I am your AI Cloud Economist. Ask me anything about your AWS cost trends, budget anomalies, or optimization strategies.",
      timestamp: new Date()
    }
  ]);
  const [input, setInput] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const scrollRef = useRef<HTMLDivElement>(null);

  // Auto-scroll to the bottom when new messages arrive
  useEffect(() => {
    if (scrollRef.current) {
      scrollRef.current.scrollIntoView({ behavior: 'smooth' });
    }
  }, [messages, isLoading]);

  const handleSendMessage = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!input.trim() || isLoading) return;

    const userMessage: Message = {
      id: crypto.randomUUID(),
      role: 'user',
      content: input,
      timestamp: new Date()
    };

    setMessages(prev => [...prev, userMessage]);
    setInput('');
    setIsLoading(true);

    try {
      // Pass history along with user query to the server action
      const reply = await askFinOpsBot(input, messages.map(m => ({ role: m.role, content: m.content })));
      
      const botMessage: Message = {
        id: crypto.randomUUID(),
        role: 'assistant',
        content: reply,
        timestamp: new Date()
      };
      setMessages(prev => [...prev, botMessage]);
    } catch (error) {
      setMessages(prev => [...prev, {
        id: crypto.randomUUID(),
        role: 'assistant',
        content: "Sorry, I ran into an error pulling your billing intelligence. Please try again.",
        timestamp: new Date()
      }]);
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <Card className="w-full h-[500px] flex flex-col col-span-4">
      <CardHeader className="pb-3 border-b">
        <div className="flex items-center gap-2">
          <Sparkles className="h-5 w-5 text-indigo-500 fill-indigo-500" />
          <div>
            <CardTitle className="text-lg">Cloud FinOps Copilot</CardTitle>
            <CardDescription>Conversational AI trained on your active AWS billing architecture</CardDescription>
          </div>
        </div>
      </CardHeader>

      <CardContent className="flex-1 overflow-y-auto p-4">
        <ScrollArea className="h-full pr-3">
          <div className="space-y-4">
            {messages.map((msg) => (
              <div key={msg.id} className={`flex gap-3 text-sm ${msg.role === 'user' ? 'justify-end' : 'justify-start'}`}>
                {msg.role === 'assistant' && (
                  <div className="h-8 w-8 rounded-full bg-indigo-100 dark:bg-indigo-950 flex items-center justify-center border shrink-0">
                    <Bot className="h-4 w-4 text-indigo-600 dark:text-indigo-400" />
                  </div>
                )}
                <div className={`p-3 rounded-lg max-w-[85%] whitespace-pre-line leading-relaxed ${
                  msg.role === 'user' 
                    ? 'bg-primary text-primary-foreground rounded-br-none' 
                    : 'bg-muted rounded-bl-none'
                }`}>
                  {msg.content}
                </div>
                {msg.role === 'user' && (
                  <div className="h-8 w-8 rounded-full bg-muted flex items-center justify-center border shrink-0">
                    <User className="h-4 w-4" />
                  </div>
                )}
              </div>
            ))}
            
            {isLoading && (
              <div className="flex gap-3 text-sm justify-start">
                <div className="h-8 w-8 rounded-full bg-indigo-100 dark:bg-indigo-950 flex items-center justify-center border shrink-0 animate-pulse">
                  <Bot className="h-4 w-4 text-indigo-600 dark:text-indigo-400" />
                </div>
                <div className="p-3 rounded-lg bg-muted rounded-bl-none flex items-center gap-2 text-muted-foreground">
                  <Loader2 className="h-4 w-4 animate-spin text-indigo-600" />
                  Analyzing infrastructure data...
                </div>
              </div>
            )}
            <div ref={scrollRef} />
          </div>
        </ScrollArea>
      </CardContent>

      <CardFooter className="p-3 border-t">
        <form onSubmit={handleSendMessage} className="flex w-full items-center space-x-2">
          <Input
            value={input}
            onChange={(e) => setInput(e.target.value)}
            placeholder="e.g., Which service is causing our current budget overrun?"
            disabled={isLoading}
            className="flex-1"
          />
          <Button type="submit" size="icon" disabled={isLoading || !input.trim()}>
            <Send className="h-4 w-4" />
          </Button>
        </form>
      </CardFooter>
    </Card>
  );
} 
⚙️ The Chat Server Action (src/actions/chat.ts)
To keep this fast and simple without running a separate backend server, use a Next.js Server Action. This fetches your live AWS metrics data framework context and injects it alongside the user's input directly into the openai API payload
"use server";

import { OpenAI } from "openai";

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

export async function askFinOpsBot(userQuery: string, chatHistory: { role: 'user' | 'assistant', content: string }[]) {
  try {
    // 1. Fetch live contextual billing info (In production, replace with real SDK calls)
    const contextData = {
      currentMonthBudget: 2000.00,
      currentSpendToDate: 1420.50,
      utilizationPercentage: "71.0%",
      topOffendingServices: [
        { name: "Amazon EC2", cost: 620.40, varianceFromLastMonth: "+18%" },
        { name: "Amazon RDS", cost: 340.10, varianceFromLastMonth: "-2%" },
        { name: "Amazon S3", cost: 180.50, varianceFromLastMonth: "+4%" }
      ],
      anomaliesDetected: "EC2 instance running unattached elastic IPs costing $45/mo; instance m5.xlarge in us-east-1 idle 92% of the time."
    };

    // 2. Construct systemic behavior directives
    const systemPrompt = `You are an expert FinOps Engineer and AWS Cloud Economist. 
    You are answering questions inside a dashboard interface. Use the following context about the user's actual AWS account metrics to tailor your answer precisely:
    ----------------
    CONTEXT:
    - Monthly Budget Cap: $${contextData.currentMonthBudget}
    - Total Spend MTD: $${contextData.currentSpendToDate} (${contextData.utilizationPercentage} consumed)
    - Top Services: ${JSON.stringify(contextData.topOffendingServices)}
    - Active Anomalies: ${contextData.anomaliesDetected}
    ----------------
    Keep responses concise, conversational, and technical. Reference specific numbers, regions, or services provided in the context data to answer queries accurately. Always format response items using markdown list syntax.`;

    // 3. Request completion from OpenAI
    const response = await openai.chat.completions.create({
      model: "gpt-4o-mini", // Cost-efficient and fast
      messages: [
        { role: "system", content: systemPrompt },
        ...chatHistory,
        { role: "user", content: userQuery }
      ],
      temperature: 0.2, // Low temperature for factual consistency on financial data
    });

    return response.choices[0].message.content || "I am unable to interpret that metric context right now.";
  } catch (error) {
    console.error("OpenAI Bot Error: ", error);
    throw new Error("Chat completion processing crashed");
  }
} 
Implementing a fully responsive layout with seamless dark/light theme switching is the final touch needed to make your Next.js FinOps application production-ready. In the Next.js ecosystem, the standard approach is combining Tailwind CSS with Next-Themes.

Here is the step-by-step setup and the code to wrap your entire layout cleanly.

🛠️ Step 1: Install Dependencies
Run this command in your project terminal:

Bash
npm install next-themes lucide-react
🌓 Step 2: Create the Theme Provider Component
Create a wrapper file at src/components/theme-provider.tsx. This configuration avoids flash-of-unstyled-content (FOUC) by injecting the theme class onto the HTML element on the server side.

TypeScript
"use client";

import * as React from "react";
import { ThemeProvider as NextThemesProvider } from "next-themes";

export function ThemeProvider({
  children,
  ...props
}: React.ComponentProps<typeof NextThemesProvider>) {
  return <NextThemesProvider {...props}>{children}</NextThemesProvider>;
}
🏗️ Step 3: Integrate with Root Layout
Wrap your main layout in src/app/layout.tsx with the provider. Make sure you set suppressHydrationWarning on the <html> tag as required by Next-Themes.

TypeScript
import "@/app/globals.css";
import { ThemeProvider } from "@/components/theme-provider";

export default function RootLayout({
  children,
  }: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body className="min-h-screen bg-background font-sans antialiased">
        <ThemeProvider
          attribute="class"
          defaultTheme="system"
          enableSystem
          disableTransitionOnChange
        >
          {children}
        </ThemeProvider>
      </body>
    </html>
  );
}
🎛️ Step 4: The Theme Toggle Switch Component
This creates a clean button that shifts back and forth between light, dark, and system default modes.

TypeScript
"use client";

import * as React from "react";
import { Moon, Sun } from "lucide-react";
import { useTheme } from "next-themes";
import { Button } from "@/components/ui/button";

export function ThemeToggle() {
  const { theme, setTheme } = useTheme();

  return (
    <Button
      variant="outline"
      size="icon"
      onClick={() => setTheme(theme === "dark" ? "light" : "dark")}
      title="Toggle theme"
    >
      <Sun className="h-[1.2rem] w-[1.2rem] rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0" />
      <Moon className="absolute h-[1.2rem] w-[1.2rem] rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100" />
      <span className="sr-only">Toggle theme</span>
    </Button>
  );
}
📱 Step 5: Responsive Dashboard Shell (src/app/page.tsx)
This layout puts all your components (Budget Cards, Cost Charts, Alerts, Chatbot) together inside a mobile-responsive grid layout.

TypeScript
import BudgetCards from "@/components/BudgetCards";
import CostAnalytics from "@/components/CostCharts";
import BudgetAlerts from "@/components/BudgetAlerts";
import CostChatbot from "@/components/CostChatbot";
import { ThemeToggle } from "@/components/ThemeToggle";
import { Cloud } from "lucide-react";

export default function DashboardPage() {
  // Mock Data definitions (can be pulled from async server actions)
  const monthlyBudget = 2000.00;
  const currentSpend = 1420.50;

  return (
    <div className="flex flex-col min-h-screen">
      {/* Responsive Navigation Header */}
      <header className="sticky top-0 z-50 w-full border-b bg-background/95 backdrop-blur supports-[backdrop-filter]:bg-background/60">
        <div className="container flex h-16 items-center justify-between px-4 md:px-8">
          <div className="flex items-center space-x-2">
            <Cloud className="h-6 w-6 text-primary" />
            <span className="font-bold inline-block text-lg">AWS FinOps AI</span>
          </div>
          <ThemeToggle />
        </div>
      </header>

      {/* Main Responsive Grid Layout */}
      <main className="flex-1 space-y-6 p-4 md:p-8 pt-6 max-w-[1600px] mx-auto w-full">
        <div className="flex items-center justify-between space-y-2">
          <h1 className="text-3xl font-bold tracking-tight">Cloud Spending Intel</h1>
        </div>

        {/* 1. High Level Metrics Group (1 Column on Mobile, 4 Columns on Large Screens) */}
        <BudgetCards monthlyBudget={monthlyBudget} currentSpend={currentSpend} />

        {/* 2. Main Analytics & Chatbot Breakdowns */}
        <div className="grid gap-6 grid-cols-1 xl:grid-cols-8">
          
          {/* Charts Window (Takes 5 columns of space out of 8 on wide displays) */}
          <div className="xl:col-span-5 space-y-6">
            <CostAnalytics />
            <BudgetAlerts monthlyBudget={monthlyBudget} currentSpend={currentSpend} />
          </div>

          {/* AI Assistant Chat Panel (Takes remaining 3 columns out of 8) */}
          <div className="xl:col-span-3">
            <CostChatbot />
          </div>

        </div>
      </main>
    </div>
  );
} 
To provide a truly professional user experience (UX) and ensure your application handles network latencies or API failures gracefully, you need explicit loading skeleton screens and robust error boundaries.

In Next.js, the cleanest way to manage this is using native Suspense boundaries, custom loading.tsx files, and local state management for asynchronous actions like the OpenAI chat or AI insights generator.

⏳ 1. Loading States (Skeleton Components)
Instead of a generic spinner that disorients the user, use Skeleton screens that match the exact shape of your UI cards.

First, ensure you have a simple skeleton utility component (or use Shadcn's layout skeleton) 
// src/components/ui/skeleton.tsx
export function Skeleton({ className, ...props }: React.HTMLAttributes<HTMLDivElement>) {
  return (
    <div
      className={`animate-pulse rounded-md bg-muted/60 ${className}`}
      {...props}
    />
  );
} 
Dashboard Skeleton Loader (src/app/loading.tsx)
Next.js automatically renders this file while your server components (async/await fetching from AWS Cost Explorer) are resolving data.
import { Skeleton } from "@/components/ui/skeleton";
import { Card, CardContent, CardHeader } from "@/components/ui/card";

export default function DashboardLoading() {
  return (
    <div className="flex flex-col min-h-screen space-y-6 p-4 md:p-8 max-w-[1600px] mx-auto w-full">
      <Skeleton className="h-9 w-48" /> {/* Page Title */}

      {/* 4 Metrics Cards Skeleton */}
      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-4">
        {[...Array(4)].map((_, i) => (
          <Card key={i}>
            <CardHeader className="pb-2"><Skeleton className="h-4 w-24" /></CardHeader>
            <CardContent>
              <Skeleton className="h-8 w-32 mb-2" />
              <Skeleton className="h-3 w-20" />
            </CardContent>
          </Card>
        ))}
      </div>

      {/* Main Analytics Layout Skeleton */}
      <div className="grid gap-6 grid-cols-1 xl:grid-cols-8">
        <div className="xl:col-span-5 space-y-6">
          {/* Chart Skeleton */}
          <Card className="h-[450px] p-6 flex flex-col justify-between">
            <div className="space-y-2">
              <Skeleton className="h-5 w-32" />
              <Skeleton className="h-4 w-48" />
            </div>
            <Skeleton className="h-[280px] w-full" />
          </Card>
        </div>
        <div className="xl:col-span-3">
          {/* Chatbot Skeleton */}
          <Card className="h-[500px]" />
        </div>
      </div>
    </div>
  );
} 
⚠️ 2. Comprehensive Error States
When integration layers depend on external networks like AWS IAM authorization and OpenAI credit balances, structural fallback states are non-negotiable.

Localized API Component Fallback (src/components/ErrorBoundaryCard.tsx)
If the AWS SDK fails to authenticate due to expired keys, don't let the whole app crash. Wrap it in a localized fallback handler 
"use client";

import { AlertOctagon, RefreshCw } from "lucide-react";
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from "@/components/ui/card";

interface ErrorCardProps {
  title: string;
  errorMsg: string;
  onRetry: () => void;
}

export default function ErrorBoundaryCard({ title, errorMsg, onRetry }: ErrorCardProps) {
  return (
    <Card className="border-destructive/50 bg-destructive/5 dark:bg-destructive/10">
      <CardHeader className="flex flex-row items-center gap-3 space-y-0">
        <div className="p-2 bg-destructive/10 rounded-full text-destructive">
          <AlertOctagon className="h-5 w-5" />
        </div>
        <div>
          <CardTitle className="text-lg text-destructive">{title}</CardTitle>
          <CardDescription>Data integration failure occurred</CardDescription>
        </div>
      </CardHeader>
      <CardContent className="space-y-4">
        <p className="text-sm text-muted-foreground bg-background p-3 rounded border font-mono">
          {errorMsg || "AWS_ACCESS_KEY_ID invalid or ce:GetCostAndUsage permission denied."}
        </p>
        <Button variant="outline" size="sm" onClick={onRetry} className="gap-2">
          <RefreshCw className="h-3 w-3" />
          Retry Request
        </Button>
      </CardContent>
    </Card>
  );
} 
🤖 3. Action State Handler for OpenAI Calls
When the user clicks the "Generate AI Insights" button, toggle an internal component state to provide crystal-clear tracking context
"use client";

import React, { useState } from 'react';
import { Button } from "@/components/ui/button";
import { Sparkles, Loader2, AlertCircle } from "lucide-react";
import ErrorBoundaryCard from './ErrorBoundaryCard';

export function AiInsightsWidget() {
  const [insights, setInsights] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const handleFetchInsights = async () => {
    setLoading(true);
    setError(null);
    try {
      // Trigger Next.js Server Action
      // const res = await getOpenAiInsights(mockBillingData);
      // setInsights(res);
    } catch (err: any) {
      setError("OpenAI API Key quota limit reached or connection timed out.");
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="space-y-4">
      {error && (
        <ErrorBoundaryCard 
          title="AI Insights Generation Error" 
          errorMsg={error} 
          onRetry={handleFetchInsights} 
        />
      )}

      {!insights && !error && (
        <Button onClick={handleFetchInsights} disabled={loading} className="w-full">
          {loading ? (
            <>
              <Loader2 className="mr-2 h-4 w-4 animate-spin" />
              Parsing Bill via GPT-4o...
            </>
          ) : (
            <>
              <Sparkles className="mr-2 h-4 w-4 fill-white" />
              Generate AI Cost Insights
            </>
          )}
        </Button>
      )}

      {loading && (
        <div className="p-6 border border-dashed rounded-xl flex flex-col items-center justify-center space-y-3 bg-muted/20">
          <Loader2 className="h-6 w-6 animate-spin text-primary" />
          <p className="text-sm text-muted-foreground animate-pulse">Running architectural analysis on cloud usage records...</p>
        </div>
      )}
    </div>
  );
} 
With the full scope established—from responsive components to theme management, AI chatbot capabilities, and rigorous loading/error handling—it is time to tie everything together.

Here is the final README.md file, your .env.example configuration, and an explicit GitHub Repository Setup roadmap optimized to secure full marks on your deliverables.

🔑 Complete .env.example File
Create a file named exactly .env.example in your root directory. Copy and paste the configuration below. It ensures reviewers know exactly what parameters your AWS SDK and OpenAI clients require to spin up the application:
# =========================================================================
# AWS ENVIRONMENT VARIABLES
# Requirements: IAM programmatic user access keys with 'ce:*' & 'budgets:*'
# =========================================================================
AWS_ACCESS_KEY_ID=your_aws_access_key_id_placeholder
AWS_SECRET_ACCESS_KEY=your_aws_secret_access_key_placeholder
AWS_REGION=us-east-1

# =========================================================================
# OPENAI AI MODELS INTEL
# =========================================================================
OPENAI_API_KEY=sk-proj-your_openai_api_token_here

# =========================================================================
# APP SYSTEM LEVEL VARIABLES
# =========================================================================
NEXT_PUBLIC_APP_URL=http://localhost:3000)))
# Set to 'true' to bypass live AWS calls and use realistic offline JSON files
NEXT_PUBLIC_USE_MOCK_DATA=false
📝 Complete README.md File
This document structures your repository professionally and provides straightforward onboarding instructions for anyone pulling down your code
# AWS Cloud FinOps Intelligence Monitor ☁️🤖

A responsive web application built with **React/Next.js**, **TypeScript**, **Tailwind CSS**, and **AWS & OpenAI APIs** to monitor cloud expenditure, visualize trends, and provide dynamic AI-powered budget optimizations.

## 🔗 Project Links
- **Live Deployment:** [Insert Vercel URL here]
- **3-5 Minute Demo Video:** [Insert Loom/Tella/YouTube URL here]

---

## ✨ Features

### 💵 Budget Overview & Alerts
- Real-time tracking of **Monthly Budget**, **Current Spend**, **Remaining Budget**, and **Utilization Rate**.
- Dynamic threshold indicators (Warning banners at 80%, Critical/Breach warnings at 100%+).

### 📊 Cost Analytics Visualizations
- Interactive multi-granularity historical analytics using `Recharts`.
- Switch seamlessly between **Daily**, **Weekly**, **Monthly**, and **Service-wise** spending categories.

### 💬 Cloud FinOps Copilot (AI Chatbot)
- Context-aware chatbot powered by `gpt-4o-mini`.
- Secure server actions feed structural AWS JSON data matrices into background system prompts for personalized recommendations.

### 🌓 UI Excellence, Performance, & Safety
- Full dark and light themes using `next-themes` with zero transition layout flash.
- Comprehensive user loading skeletons via Next.js Suspense architecture.
- Modular component-level error boundaries with integrated retry execution fallback loops.

---

## 🛠️ Technical Stack
- **Framework:** Next.js 14+ (App Router)
- **Language:** TypeScript
- **Styling:** Tailwind CSS + Shadcn UI
- **Data Graphs:** Recharts
- **SDKs:** `@aws-sdk/client-cost-explorer`, `@aws-sdk/client-budgets`, `openai`

---

## ⚙️ Getting Started & Installation

### Prerequisites
- Node.js v18 or higher installed.
- An AWS IAM user possessing programmatic access permissions (`ce:GetCostAndUsage`, `ce:GetCostForecast`, `budgets:ViewBudget`).
- An active OpenAI developer API token.

### Setup Instructions

1. **Clone the repository:**
   ```bash
   git clone [https://github.com/your-username/aws-finops-monitor.git](https://github.com/your-username/aws-finops-monitor.git)
   cd aws-finops-monitor
   Install local dependencies:

Bash
npm install
Establish Environment Variables:

Bash
cp .env.example .env
Open the newly created .env file and append your valid AWS and OpenAI tokens.

Boot Up Local Development Server:

Bash
npm run dev
Open your browser to http://localhost:3000 to view the application.

💡 How It Works (Data Architecture)
Extraction: Next.js Server Actions fetch live cloud financial metrics via the official @aws-sdk/client-cost-explorer.

Parsing & Rendering: Server data feeds directly into structured Recharts coordinate arrays.

AI Enhancement Engine: When launching a chat or requesting insights, the application contextually serializes active structural spending datasets into localized OpenAI API completions. This keeps private application keys hidden on the server, ensuring they are never exposed to client browsers
---

## 🚀 Final Delivery Checklist

Before recording your video and handing in the links, double-check that your work meets these standards:

- [ ] **GitHub Repository:** Double-check your `.gitignore` to ensure your actual `.env` file is hidden and not committed to history.
- [ ] **Vercel Settings:** Go to your project settings in the Vercel Dashboard and input your `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `OPENAI_API_KEY` environmental parameters so the live environment can compile and pull data successfully.
- [ ] **Mock Toggle Verification:** If you do not want to connect your live production billing credentials to Vercel, verify that your codebase handles `NEXT_PUBLIC_USE_MOCK_DATA=true` smoothly. This ensures reviewers can interact with realistic data charts and chatbot flows without needing your live AWS login credentials.
