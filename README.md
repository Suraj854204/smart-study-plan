# smart-study-plan
frontent code--react

import { useQuery } from "@tanstack/react-query";
import { SubjectWithTopics } from "@shared/schema";
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from "@/components/ui/card";
import { differenceInDays, format, parseISO } from "date-fns";

const Exams = () => {
  const { data: subjects, isLoading } = useQuery<SubjectWithTopics[]>({
    queryKey: ["/api/subjects-with-topics"],
  });
  
  // Filter subjects with exam dates and sort by date (closest first)
  const upcomingExams = subjects
    ?.filter(subject => subject.examDate)
    .sort((a, b) => {
      if (!a.examDate || !b.examDate) return 0;
      return new Date(a.examDate).getTime() - new Date(b.examDate).getTime();
    });
  
  // Group exams by month for better organization
  const examsByMonth: Record<string, SubjectWithTopics[]> = {};
  
  upcomingExams?.forEach(subject => {
    if (!subject.examDate) return;
    
    const date = new Date(subject.examDate);
    const monthYear = format(date, 'MMMM yyyy');
    
    if (!examsByMonth[monthYear]) {
      examsByMonth[monthYear] = [];
    }
    
    examsByMonth[monthYear].push(subject);
  });
  
  // Function to determine urgency level
  const getUrgencyLevel = (examDate: Date) => {
    const daysUntilExam = differenceInDays(new Date(examDate), new Date());
    
    if (daysUntilExam <= 7) return "high";
    if (daysUntilExam <= 14) return "medium";
    return "low";
  };
  
  // Function to get urgency color
  const getUrgencyColor = (examDate: Date) => {
    const urgency = getUrgencyLevel(examDate);
    
    switch (urgency) {
      case "high": return "text-red-500";
      case "medium": return "text-orange-500";
      case "low": return "text-green-500";
      default: return "text-gray-500";
    }
  };
  
  // Function to format days until exam
  const formatDaysUntil = (examDate: Date) => {
    const daysUntilExam = differenceInDays(new Date(examDate), new Date());
    
    if (daysUntilExam === 0) return "Today";
    if (daysUntilExam === 1) return "Tomorrow";
    return `${daysUntilExam} days`;
  };

  return (
    <main className="flex-grow p-4 bg-gray-50 overflow-auto">
      <div className="container mx-auto">
        <div className="mb-6">
          <h1 className="text-2xl font-bold text-gray-900">Upcoming Exams</h1>
          <p className="text-gray-500 mt-1">View and manage your exam schedule.</p>
        </div>
        
        {isLoading ? (
          <div className="flex justify-center py-20">
            <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-primary"></div>
          </div>
        ) : !upcomingExams || upcomingExams.length === 0 ? (
          <div className="text-center py-20 bg-white rounded-lg shadow">
            <svg xmlns="http://www.w3.org/2000/svg" className="h-16 w-16 mx-auto text-gray-400 mb-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="1" strokeLinecap="round" strokeLinejoin="round">
              <rect x="3" y="4" width="18" height="18" rx="2" ry="2" />
              <line x1="16" y1="2" x2="16" y2="6" />
              <line x1="8" y1="2" x2="8" y2="6" />
              <line x1="3" y1="10" x2="21" y2="10" />
            </svg>
            <h2 className="text-xl font-semibold text-gray-700 mb-2">No Exams Scheduled</h2>
            <p className="text-gray-500 max-w-md mx-auto">
              You haven't set any exam dates yet. Add exam dates to your subjects to see them here.
            </p>
          </div>
        ) : (
          <div className="space-y-8">
            {Object.entries(examsByMonth).map(([monthYear, exams]) => (
              <div key={monthYear}>
                <h2 className="text-xl font-semibold text-gray-800 mb-4">{monthYear}</h2>
                <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                  {exams.map(subject => (
                    <Card key={subject.id} className="flex">
                      <div 
                        className="w-2 flex-shrink-0"
                        style={{ backgroundColor: subject.color }}
                      ></div>
                      <div className="flex-grow">
                        <CardHeader className="pb-2">
                          <CardTitle className="flex justify-between">
                            <span>{subject.name}</span>
                            {subject.examDate && (
                              <span className={`text-sm font-normal ${getUrgencyColor(subject.examDate)}`}>
                                {formatDaysUntil(subject.examDate)}
                              </span>
                            )}
                          </CardTitle>
                          <CardDescription>
                            {subject.examDate && format(new Date(subject.examDate), 'EEEE, MMMM d, yyyy')}
                          </CardDescription>
                        </CardHeader>
                        <CardContent>
                          <div className="flex justify-between text-sm">
                            <span className="text-gray-500">Difficulty:</span>
                            <span className="font-medium">{subject.difficulty}</span>
                          </div>
                          <div className="flex justify-between text-sm mt-1">
                            <span className="text-gray-500">Topics:</span>
                            <span className="font-medium">{subject.topics.length}</span>
                          </div>
                          <div className="flex justify-between text-sm mt-1">
                            <span className="text-gray-500">Completed:</span>
                            <span className="font-medium">
                              {subject.topics.filter(t => t.completed).length}/{subject.topics.length}
                            </span>
                          </div>
                        </CardContent>
                      </div>
                    </Card>
                  ))}
                </div>
              </div>
            ))}
          </div>
        )}
      </div>
    </main>
  );
};

export default Exams;



backend -code python

import { SubjectWithTopics } from "@shared/schema";
const PYTHON_API_BASE_URL = "/python-api";

// Log the API URL being used
console.log("Python scheduler API URL:", PYTHON_API_BASE_URL);

export async function generateScheduleWithPython(
  subjects: SubjectWithTopics[],
  dailyHours: number
): Promise<any> {
  try {
    // Try to use the Python API
    try {
      console.log("Attempting to connect to Python scheduler API...");
      const response = await fetch(`${PYTHON_API_BASE_URL}/generate-schedule`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          subjects,
          dailyHours,
        }),
      });

      if (!response.ok) {
        const errorData = await response.json();
        throw new Error(errorData.error || "Failed to generate schedule");
      }

      const result = await response.json();
      console.log("Successfully generated schedule using Python API");
      return result;
    } catch (apiError) {
      console.warn("Could not connect to Python scheduler API, using client-side fallback method", apiError);
      
      // Create a simple schedule if Python API is unavailable
      const daysOfWeek = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"];
      const timeSlots = ["09:00", "10:00", "11:00", "13:00", "14:00", "15:00", "16:00", "17:00"];
      
      // Calculate priorities for subjects
      const subjectsWithPriorities = subjects.map(subject => ({
        ...subject,
        priority: calculateSimplePriority(subject)
      })).sort((a, b) => b.priority - a.priority);
      
      const schedule = [];
      let subjectIndex = 0;
      let topicIndex = 0;
      
      // Generate simple round-robin schedule
      for (const day of daysOfWeek) {
        for (const time of timeSlots) {
          // Skip if all subjects are processed
          if (subjectsWithPriorities.length === 0) continue;
          
          // Get current subject and rotate
          if (subjectIndex >= subjectsWithPriorities.length) {
            subjectIndex = 0;
          }
          
          const currentSubject = subjectsWithPriorities[subjectIndex];
          const topics = currentSubject.topics.filter(t => !t.completed);
          
          // Skip if no uncompleted topics
          if (topics.length > 0) {
            if (topicIndex >= topics.length) {
              topicIndex = 0;
            }
            
            const currentTopic = topics[topicIndex];
            
            // Add slot to schedule
            schedule.push({
              day,
              startTime: time,
              subjectId: currentSubject.id,
              topicId: currentTopic.id
            });
            
            topicIndex++;
          }
          
          subjectIndex++;
        }
      }
      
      return {
        schedule,
        dailyHours,
        generatedAt: new Date().toISOString(),
        metadata: {
          generatedBy: "client-fallback",
          subjects: subjectsWithPriorities
        }
      };
    }
  } catch (error) {
    console.error("Error generating schedule:", error);
    throw error;
  }
}

/**
 * Calculate priorities for subjects using the Python scheduler API
 */
export async function calculatePriorities(subjects: SubjectWithTopics[]) {
  try {
    // Try to use the Python API
    try {
      const response = await fetch(`${PYTHON_API_BASE_URL}/calculate-priorities`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          subjects,
        }),
      });

      if (!response.ok) {
        const errorData = await response.json();
        throw new Error(errorData.error || "Failed to calculate priorities");
      }

      return response.json();
    } catch (apiError) {
      console.warn("Could not connect to Python scheduler API, using fallback method", apiError);
      
      // Use the fallback method if the Python API is unavailable
      const priorities = subjects.map(subject => ({
        id: subject.id,
        name: subject.name,
        priority_score: calculateSimplePriority(subject)
      }));
      
      // Sort by priority (highest first)
      priorities.sort((a, b) => b.priority_score - a.priority_score);
      
      return { priorities };
    }
  } catch (error) {
    console.error("Error calculating priorities:", error);
    throw error;
  }
}

/**
 * Simple priority calculation as a fallback method
 */
function calculateSimplePriority(subject: SubjectWithTopics): number {
  // Default medium priority
  let priority = 50;
  
  // If there's an exam date, increase priority as date approaches
  if (subject.examDate) {
    const today = new Date();
    const examDate = new Date(subject.examDate);
    const daysUntilExam = Math.max(1, Math.floor((examDate.getTime() - today.getTime()) / (1000 * 60 * 60 * 24)));
    
    // Exponentially increase priority as exam gets closer
    if (daysUntilExam <= 7) {
      // Within a week: highest priority
      priority = 100;
    } else if (daysUntilExam <= 14) {
      // Within two weeks: high priority
      priority = 80;
    } else if (daysUntilExam <= 30) {
      // Within a month: medium-high priority
      priority = 65;
    } else {
      // More than a month away: medium-low priority
      priority = 40;
    }
  }
  
  // Adjust based on difficulty
  if (subject.difficulty === "hard") {
    priority += 15;
  } else if (subject.difficulty === "easy") {
    priority -= 10;
  }
  
  // Adjust based on topic completion
  const totalTopics = subject.topics.length;
  if (totalTopics > 0) {
    const completedTopics = subject.topics.filter(t => t.completed).length;
    const completionRatio = completedTopics / totalTopics;
    
    // Lower priority if mostly completed, higher if mostly incomplete
    priority = priority * (1 + (0.5 - completionRatio) / 2);
  }
  
  // Ensure priority is within bounds
  return Math.max(1, Math.min(100, Math.round(priority)));
}

