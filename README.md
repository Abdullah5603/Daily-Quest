<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Daily Challenges Hub</title>
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@fortawesome/fontawesome-free@6.4.0/css/all.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/alpinejs@3.12.0/dist/cdn.min.js" defer></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body {
            font-family: 'Poppins', sans-serif;
            background-color: #f0f2f5;
        }
        .challenge-card {
            border-radius: 12px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            transition: all 0.3s ease;
        }
        .challenge-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 15px rgba(0, 0, 0, 0.1);
        }
        .badge {
            width: 50px;
            height: 50px;
            border-radius: 50%;
            background: linear-gradient(45deg, #6366f1, #8b5cf6);
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-size: 20px;
            margin-right: 10px;
            position: relative;
        }
        .badge-locked {
            background: linear-gradient(45deg, #9ca3af, #d1d5db);
            opacity: 0.7;
        }
        .badge::after {
            content: attr(data-tooltip);
            position: absolute;
            bottom: -30px;
            left: 50%;
            transform: translateX(-50%);
            padding: 4px 8px;
            border-radius: 4px;
            background-color: #1f2937;
            color: white;
            font-size: 12px;
            white-space: nowrap;
            visibility: hidden;
            opacity: 0;
            transition: all 0.3s ease;
        }
        .badge:hover::after {
            visibility: visible;
            opacity: 1;
        }
        .letter-tile {
            width: 50px;
            height: 50px;
            border: 2px solid #d1d5db;
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: bold;
            font-size: 24px;
            text-transform: uppercase;
        }
        .tile-correct {
            background-color: #10b981;
            color: white;
            border-color: #10b981;
        }
        .tile-present {
            background-color: #f59e0b;
            color: white;
            border-color: #f59e0b;
        }
        .tile-absent {
            background-color: #6b7280;
            color: white;
            border-color: #6b7280;
        }
        .keyboard-row {
            display: flex;
            justify-content: center;
            margin-bottom: 8px;
        }
        .key {
            min-width: 40px;
            height: 58px;
            margin: 0 4px 0 0;
            border-radius: 4px;
            display: flex;
            align-items: center;
            justify-content: center;
            background-color: #d1d5db;
            cursor: pointer;
            text-transform: uppercase;
            font-weight: bold;
        }
        .streak-counter {
            font-size: 18px;
            font-weight: bold;
            color: #4f46e5;
            padding: 8px 12px;
            background-color: rgba(79, 70, 229, 0.1);
            border-radius: 20px;
        }
        .tab-button {
            padding: 10px 15px;
            background-color: #f3f4f6;
            border-radius: 8px 8px 0 0;
            cursor: pointer;
            transition: background-color 0.3s ease;
        }
        .tab-button.active {
            background-color: white;
            font-weight: 600;
        }
        .progress-ring {
            transform: rotate(-90deg);
        }
        .progress-ring__circle {
            stroke-dasharray: 283;
            transition: stroke-dashoffset 0.3s ease;
        }
        #confetti-canvas {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: 9999;
        }
    </style>
</head>
<body x-data="app()" class="min-h-screen pb-20">
    <canvas id="confetti-canvas"></canvas>
    
    <header class="bg-gradient-to-r from-indigo-600 to-purple-600 text-white py-6">
        <div class="container mx-auto px-4">
            <div class="flex flex-col md:flex-row justify-between items-center">
                <div>
                    <h1 class="text-3xl font-bold mb-2">Daily Challenges Hub</h1>
                    <p class="text-indigo-100">Complete daily challenges, earn badges, climb the leaderboard!</p>
                </div>
                <div class="mt-4 md:mt-0 flex items-center space-x-4">
                    <div class="bg-white bg-opacity-20 rounded-lg p-3">
                        <div class="text-sm">Today's Date</div>
                        <div class="font-bold" x-text="getCurrentDate()"></div>
                    </div>
                    <div class="bg-white bg-opacity-20 rounded-lg p-3">
                        <div class="text-sm">Next Refresh</div>
                        <div class="font-bold">24:00:00</div>
                    </div>
                </div>
            </div>
            
            <div class="mt-8 flex flex-wrap justify-center gap-2">
                <div class="streak-counter">
                    <i class="fas fa-fire"></i> Current Streak: <span x-text="userStats.currentStreak"></span> days
                </div>
                <div class="streak-counter">
                    <i class="fas fa-trophy"></i> Total Points: <span x-text="userStats.totalPoints"></span>
                </div>
                <div class="streak-counter">
                    <i class="fas fa-star"></i> Rank: <span x-text="userStats.rank"></span>
                </div>
            </div>
        </div>
    </header>

    <div class="container mx-auto px-4 py-8">
        <!-- User Dashboard -->
        <div class="bg-white rounded-lg shadow-md p-6 mb-8">
            <h2 class="text-2xl font-semibold mb-4">Your Progress</h2>
            
            <div class="grid grid-cols-1 md:grid-cols-5 gap-4 mb-6">
                <template x-for="(challenge, index) in challenges" :key="index">
                    <div class="flex items-center">
                        <svg class="progress-ring" width="60" height="60">
                            <circle
                                class="progress-ring__circle bg-gray-200"
                                stroke="#e5e7eb"
                                stroke-width="4"
                                fill="transparent"
                                r="25"
                                cx="30"
                                cy="30"
                            />
                            <circle
                                class="progress-ring__circle"
                                :stroke="challenge.completed ? '#10b981' : '#6366f1'"
                                stroke-width="4"
                                fill="transparent"
                                r="25"
                                cx="30"
                                cy="30"
                                :stroke-dashoffset="challenge.completed ? 0 : 283"
                            />
                        </svg>
                        <div class="ml-3">
                            <div class="font-medium" x-text="challenge.name"></div>
                            <div class="text-sm text-gray-500" x-text="challenge.completed ? 'Completed' : 'Pending'"></div>
                        </div>
                    </div>
                </template>
            </div>
            
            <h3 class="text-xl font-semibold mb-2">Your Badges</h3>
            <div class="flex flex-wrap gap-4 mb-6">
                <div class="badge" data-tooltip="5-Day Streak">
                    <i class="fas fa-fire"></i>
                </div>
                <div class="badge" data-tooltip="Word Master">
                    <i class="fas fa-spell-check"></i>
                </div>
                <div class="badge" data-tooltip="Trivia Expert">
                    <i class="fas fa-brain"></i>
                </div>
                <div class="badge badge-locked" data-tooltip="Creative Writer">
                    <i class="fas fa-pen-fancy"></i>
                </div>
                <div class="badge badge-locked" data-tooltip="Emoji Translator">
                    <i class="far fa-grin"></i>
                </div>
                <div class="badge badge-locked" data-tooltip="Artist">
                    <i class="fas fa-palette"></i>
                </div>
                <div class="badge badge-locked" data-tooltip="Perfect Week">
                    <i class="fas fa-calendar-check"></i>
                </div>
            </div>
            
            <button @click="showReminderModal = true" class="bg-indigo-600 hover:bg-indigo-700 text-white font-medium py-2 px-4 rounded-lg transition">
                <i class="far fa-bell mr-2"></i> Set Daily Reminder
            </button>
        </div>

        <!-- Challenge Tabs -->
        <div class="flex flex-wrap border-b border-gray-200 mb-6">
            <template x-for="(challenge, index) in challenges" :key="index">
                <div 
                    class="tab-button"
                    :class="activeTab === index ? 'active' : ''"
                    @click="setActiveTab(index)"
                    x-text="challenge.name"
                ></div>
            </template>
        </div>

        <!-- Word Guess Challenge -->
        <div x-show="activeTab === 0" class="challenge-card bg-white p-6">
            <div class="flex justify-between items-start mb-6">
                <div>
                    <h2 class="text-2xl font-bold mb-2">Word Guess</h2>
                    <p class="text-gray-600">Guess today's 5-letter word in 6 tries or less.</p>
                </div>
                <span class="bg-indigo-100 text-indigo-800 text-xs font-semibold px-2.5 py-0.5 rounded">Points: 50</span>
            </div>
            
            <div class="flex flex-col items-center mb-8" x-show="!challenges[0].completed">
                <!-- Word rows -->
                <div class="grid grid-rows-6 gap-2 mb-6">
                    <template x-for="(attempt, attemptIndex) in wordGuess.attempts" :key="attemptIndex">
                        <div class="flex gap-2">
                            <template x-for="(letter, letterIndex) in 5" :key="letterIndex">
                                <div 
                                    class="letter-tile"
                                    :class="{
                                        'tile-correct': attempt.status?.[letterIndex] === 'correct',
                                        'tile-present': attempt.status?.[letterIndex] === 'present',
                                        'tile-absent': attempt.status?.[letterIndex] === 'absent'
                                    }"
                                    x-text="attempt.word[letterIndex] || ''"
                                ></div>
                            </template>
                        </div>
                    </template>
                </div>
                
                <!-- Keyboard -->
                <div class="w-full max-w-md">
                    <div class="keyboard-row">
                        <template x-for="letter in ['q','w','e','r','t','y','u','i','o','p']" :key="letter">
                            <div class="key" @click="addLetter(letter)" x-text="letter"></div>
                        </template>
                    </div>
                    <div class="keyboard-row">
                        <template x-for="letter in ['a','s','d','f','g','h','j','k','l']" :key="letter">
                            <div class="key" @click="addLetter(letter)" x-text="letter"></div>
                        </template>
                    </div>
                    <div class="keyboard-row">
                        <div class="key px-2" style="width: 65px;" @click="submitWord()">ENTER</div>
                        <template x-for="letter in ['z','x','c','v','b','n','m']" :key="letter">
                            <div class="key" @click="addLetter(letter)" x-text="letter"></div>
                        </template>
                        <div class="key px-2" style="width: 65px;" @click="deleteLetter()">
                            <i class="fas fa-backspace"></i>
                        </div>
                    </div>
                </div>
            </div>
            
            <div x-show="challenges[0].completed" class="text-center py-8">
                <div class="text-5xl mb-4">🎉</div>
                <h3 class="text-xl font-bold mb-2">Congratulations!</h3>
                <p class="mb-2">You successfully guessed today's word: <span class="font-bold text-green-600">CHALK</span></p>
                <p>You earned 50 points and maintained your streak!</p>
            </div>
            
            <div class="bg-gray-50 p-4 rounded-lg">
                <h3 class="font-bold mb-2">Leaderboard</h3>
                <div class="space-y-2">
                    <div class="flex justify-between items-center">
                        <div class="flex items-center">
                            <span class="bg-yellow-400 h-5 w-5 rounded-full flex items-center justify-center text-xs font-bold mr-2">1</span>
                            <span>WordMaster92</span>
                        </div>
                        <div class="text-sm">2 attempts</div>
                    </div>
                    <div class="flex justify-between items-center">
                        <div class="flex items-center">
                            <span class="bg-gray-300 h-5 w-5 rounded-full flex items-center justify-center text-xs font-bold mr-2">2</span>
                            <span>LexiconLover</span>
                        </div>
                        <div class="text-sm">3 attempts</div>
                    </div>
                    <div class="flex justify-between items-center">
                        <div class="flex items-center">
                            <span class="bg-amber-700 h-5 w-5 rounded-full flex items-center justify-center text-xs text-white font-bold mr-2">3</span>
                            <span>WordWizard</span>
                        </div>
                        <div class="text-sm">3 attempts</div>
                    </div>
                    <div class="flex justify-between items-center mt-2 pt-2 border-t border-gray-200">
                        <div class="flex items-center">
                            <span class="bg-blue-500 h-5 w-5 rounded-full flex items-center justify-center text-xs text-white font-bold mr-2">
                                <i class="fas fa-user-circle text-xs"></i>
                            </span>
                            <span>You</span>
                        </div>
                        <div class="text-sm">4 attempts</div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Drawing Prompt Challenge -->
        <div x-show="activeTab === 1" class="challenge-card bg-white p-6">
            <div class="flex justify-between items-start mb-6">
                <div>
                    <h2 class="text-2xl font-bold mb-2">Drawing Prompt Generator</h2>
                    <p class="text-gray-600">Create art based on today's quirky prompt.</p>
                </div>
                <span class="bg-indigo-100 text-indigo-800 text-xs font-semibold px-2.5 py-0.5 rounded">Points: 40</span>
            </div>
            
            <div class="bg-gradient-to-r from-pink-100 to-purple-100 p-6 rounded-lg mb-6 text-center">
                <div class="text-lg font-medium mb-2">Today's Prompt:</div>
                <div class="text-2xl font-bold mb-4">"A robot trying to eat ice cream on a hot summer day"</div>
                <div class="text-sm text-gray-600">Submit your drawing based on this prompt before midnight!</div>
            </div>
            
            <div x-show="!challenges[1].completed" class="mb-6">
                <div class="border-2 border-dashed border-gray-300 rounded-lg p-8 text-center mb-4">
                    <i class="fas fa-cloud-upload-alt text-gray-400 text-3xl mb-2"></i>
                    <p class="mb-4">Drag and drop your drawing here or click to upload</p>
                    <input type="file" class="hidden" id="drawing-upload">
                    <label for="drawing-upload" class="bg-indigo-600 hover:bg-indigo-700 text-white font-medium py-2 px-4 rounded-lg transition cursor-pointer">
                        Upload Drawing
                    </label>
                </div>
                
                <div class="flex justify-end">
                    <button 
                        class="bg-indigo-600 hover:bg-indigo-700 text-white font-medium py-2 px-4 rounded-lg transition"
                        @click="completeChallenge(1)"
                    >
                        Submit Drawing
                    </button>
                </div>
            </div>
            
            <div x-show="challenges[1].completed" class="text-center py-8">
                <div class="text-5xl mb-4">🎨</div>
                <h3 class="text-xl font-bold mb-2">Submission Received!</h3>
                <p class="mb-4">Your drawing has been submitted. Check back tomorrow for community votes!</p>
                <p>You earned 40 points and maintained your streak!</p>
            </div>
            
            <div class="mt-6">
                <h3 class="font-bold mb-4">Previous Day's Winners</h3>
                <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                    <div class="bg-gray-50 p-4 rounded-lg">
                        <div class="flex justify-between items-center mb-2">
                            <div class="font-medium">1st Place</div>
                            <div class="bg-yellow-100 text-yellow-800 text-xs font-semibold px-2 py-0.5 rounded">
                                <i class="fas fa-thumbs-up mr-1"></i> 142
                            </div>
                        </div>
                        <div class="aspect-square bg-gray-200 rounded-lg flex items-center justify-center">
                            <span class="text-gray-400">Image Placeholder</span>
                        </div>
                        <div class="mt-2 text-sm text-gray-600">By: CreativeSoul</div>
                    </div>
                    <div class="bg-gray-50 p-4 rounded-lg">
                        <div class="flex justify-between items-center mb-2">
                            <div class="font-medium">2nd Place</div>
                            <div class="bg-yellow-100 text-yellow-800 text-xs font-semibold px-2 py-0.5 rounded">
                                <i class="fas fa-thumbs-up mr-1"></i> 98
                            </div>
                        </div>
                        <div class="aspect-square bg-gray-200 rounded-lg flex items-center justify-center">
                            <span class="text-gray-400">Image Placeholder</span>
                        </div>
                        <div class="mt-2 text-sm text-gray-600">By: ArtisticMind</div>
                    </div>
                    <div class="bg-gray-50 p-4 rounded-lg">
                        <div class="flex justify-between items-center mb-2">
                            <div class="font-medium">3rd Place</div>
                            <div class="bg-yellow-100 text-yellow-800 text-xs font-semibold px-2 py-0.5 rounded">
                                <i class="fas fa-thumbs-up mr-1"></i> 76
                            </div>
                        </div>
                        <div class="aspect-square bg-gray-200 rounded-lg flex items-center justify-center">
                            <span class="text-gray-400">Image Placeholder</span>
                        </div>
                        <div class="mt-2 text-sm text-gray-600">By: ColorMaster</div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Trivia Challenge -->
        <div x-show="activeTab === 2" class="challenge-card bg-white p-6">
            <div class="flex justify-between items-start mb-6">
                <div>
                    <h2 class="text-2xl font-bold mb-2">Trivia Daily</h2>
                    <p class="text-gray-600">Test your knowledge with today's random trivia question.</p>
                </div>
                <span class="bg-indigo-100 text-indigo-800 text-xs font-semibold px-2.5 py-0.5 rounded">Points: 30</span>
            </div>
            
            <div x-show="!challenges[2].completed" class="mb-8">
                <div class="bg-blue-50 p-6 rounded-lg mb-6">
                    <div class="text-sm font-medium text-blue-800 mb-1">Category: Science</div>
                    <div class="text-xl font-bold mb-4">Which planet in our solar system rotates clockwise?</div>
                </div>
                
                <div class="space-y-3">
                    <button 
                        @click="checkTriviaAnswer('Mars')"
                        class="w-full text-left p-4 border border-gray-300 rounded-lg hover:bg-gray-50 transition"
                    >
                        A. Mars
                    </button>
                    <button 
                        @click="checkTriviaAnswer('Venus')"
                        class="w-full text-left p-4 border border-gray-300 rounded-lg hover:bg-gray-50 transition"
                    >
                        B. Venus
                    </button>
                    <button 
                        @click="checkTriviaAnswer('Jupiter')"
                        class="w-full text-left p-4 border border-gray-300 rounded-lg hover:bg-gray-50 transition"
                    >
                        C. Jupiter
                    </button>
                    <button 
                        @click="checkTriviaAnswer('Uranus')"
                        class="w-full text-left p-4 border border-gray-300 rounded-lg hover:bg-gray-50 transition"
                    >
                        D. Uranus
                    </button>
                </div>
            </div>
            
            <div x-show="challenges[2].completed && triviaCorrect" class="text-center py-8">
                <div class="text-5xl mb-4">🎯</div>
                <h3 class="text-xl font-bold mb-2 text-green-600">Correct Answer!</h3>
                <p class="mb-2">Venus is the only planet in our solar system that rotates clockwise.</p>
                <p>You earned 30 points and maintained your streak!</p>
            </div>
            
            <div x-show="challenges[2].completed && !triviaCorrect" class="text-center py-8">
                <div class="text-5xl mb-4">😢</div>
                <h3 class="text-xl font-bold mb-2 text-red-600">Incorrect Answer</h3>
                <p class="mb-2">The correct answer is Venus. It's the only planet in our solar system that rotates clockwise.</p>
                <p>Better luck tomorrow!</p>
            </div>
            
            <div class="bg-gray-50 p-4 rounded-lg">
                <h3 class="font-bold mb-2">Daily Trivia Stats</h3>
                <div>
                    <canvas id="triviaChart" height="200"></canvas>
                </div>
                <div class="mt-4 text-sm text-gray-600 text-center">
                    Based on 1,234 answers today
                </div>
            </div>
        </div>

        <!-- Micro Story Challenge -->
        <div x-show="activeTab === 3" class="challenge-card bg-white p-6">
            <div class="flex justify-between items-start mb-6">
                <div>
                    <h2 class="text-2xl font-bold mb-2">Micro Story Contest</h2>
                    <p class="text-gray-600">Write a 100-word story based on today's theme.</p>
                </div>
                <span class="bg-indigo-100 text-indigo-800 text-xs font-semibold px-2.5 py-0.5 rounded">Points: 45</span>
            </div>
            
            <div class="bg-gradient-to-r from-green-100 to-teal-100 p-6 rounded-lg mb-6 text-center">
                <div class="text-lg font-medium mb-2">Today's Theme:</div>
                <div class="text-2xl font-bold mb-4">"The Last Key"</div>
                <div class="text-sm text-gray-700">Write a 100-word story inspired by this theme.</div>
            </div>
            
            <div x-show="!challenges[3].completed" class="mb-6">
                <div class="mb-4">
                    <textarea 
                        class="w-full h-40 p-4 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500"
                        placeholder="Start writing your micro story here..."
                        x-model="microStory"
                    ></textarea>
                    <div class="flex justify-between text-sm text-gray-500 mt-2">
                        <span>Max 100 words</span>
                        <span x-text="countWords(microStory) + ' / 100'"></span>
                    </div>
                </div>
                
                <div class="flex justify-end">
                    <button 
                        class="bg-indigo-600 hover:bg-indigo-700 text-white font-medium py-2 px-4 rounded-lg transition"
                        @click="completeChallenge(3)"
                        :disabled="countWords(microStory) > 100"
                    >
                        Submit Story
                    </button>
                </div>
            </div>
            
            <div x-show="challenges[3].completed" class="text-center py-8">
                <div class="text-5xl mb-4">📝</div>
                <h3 class="text-xl font-bold mb-2">Story Submitted!</h3>
                <p class="mb-4">Your micro story has been submitted. Check back tomorrow for community votes!</p>
                <p>You earned 45 points and maintained your streak!</p>
            </div>
            
            <div class="mt-6">
                <h3 class="font-bold mb-4">Previous Day's Winning Story</h3>
                <div class="bg-gray-50 p-6 rounded-lg">
                    <div class="flex justify-between items-start mb-4">
                        <h4 class="font-medium">Theme: "The Silent Forest"</h4>
                        <div class="bg-yellow-100 text-yellow-800 text-xs font-semibold px-2 py-0.5 rounded">
                            <i class="fas fa-star mr-1"></i> Winner
                        </div>
                    </div>
                    <div class="text-gray-700 mb-4 italic">
                        "Trees whispered secrets in the silent forest. I'd visited since childhood, but today was different. 
                        Air felt charged, heavy with anticipation. I followed a trail of glowing mushrooms deeper than I'd ever ventured.
                        At the heart, a clearing bathed in moonlight. A figure emerged—translucent, beautiful, terrifying.
                        'You're the first to hear us,' it said.
                        'Hear what?'
                        'The forest's dying words.'
                        I understood then why I'd been called. Not to witness magic, but to bear witness to an ending.
                        The last human who would know the forest's voice."
                    </div>
                    <div class="text-sm text-gray-500">By: WordWeaver</div>
                </div>
            </div>
        </div>

        <!-- Emoji Translation Challenge -->
        <div x-show="activeTab === 4" class="challenge-card bg-white p-6">
            <div class="flex justify-between items-start mb-6">
                <div>
                    <h2 class="text-2xl font-bold mb-2">Emoji Translation Game</h2>
                    <p class="text-gray-600">Guess the movie or song title from the emoji sequence.</p>
                </div>
                <span class="bg-indigo-100 text-indigo-800 text-xs font-semibold px-2.5 py-0.5 rounded">Points: 35</span>
            </div>
            
            <div class="bg-gradient-to-r from-yellow-100 to-orange-100 p-6 rounded-lg mb-6 text-center">
                <div class="text-lg font-medium mb-2">Today's Emoji Sequence:</div>
                <div class="text-4xl mb-4">🧙‍♂️💍🌋🧝‍♂️</div>
                <div class="text-sm text-gray-700">Guess the movie title represented by these emojis.</div>
            </div>
            
            <div x-show="!challenges[4].completed" class="mb-6">
                <div class="mb-4">
                    <input 
                        type="text" 
                        class="w-full p-4 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500"
                        placeholder="Enter your answer..."
                        x-model="emojiGuess"
                    />
                </div>
                
                <div class="flex justify-between items-center">
                    <button 
                        class="bg-gray-200 hover:bg-gray-300 text-gray-800 font-medium py-2 px-4 rounded-lg transition"
                        @click="showEmojiHint = !showEmojiHint"
                    >
                        <i class="far fa-lightbulb mr-1"></i> Hint
                    </button>
                    
                    <button 
                        class="bg-indigo-600 hover:bg-indigo-700 text-white font-medium py-2 px-4 rounded-lg transition"
                        @click="checkEmojiAnswer()"
                    >
                        Submit Answer
                    </button>
                </div>
                
                <div x-show="showEmojiHint" class="mt-4 p-4 bg-blue-50 rounded-lg text-sm">
                    <strong>Hint:</strong> A fantasy film series about a magical ring and a dangerous journey.
                </div>
            </div>
            
            <div x-show="challenges[4].completed && emojiCorrect" class="text-center py-8">
                <div class="text-5xl mb-4">🎉</div>
                <h3 class="text-xl font-bold mb-2 text-green-600">Correct!</h3>
                <p class="mb-4">The emoji sequence 🧙‍♂️💍🌋🧝‍♂️ represents "The Lord of the Rings"</p>
                <p>You earned 35 points and maintained your streak!</p>
            </div>
            
            <div x-show="challenges[4].completed && !emojiCorrect" class="text-center py-8">
                <div class="text-5xl mb-4">😢</div>
                <h3 class="text-xl font-bold mb-2 text-red-600">Incorrect</h3>
                <p class="mb-4">The correct answer is "The Lord of the Rings"</p>
                <p class="mb-2">Wizard (Gandalf) + Ring (The One Ring) + Volcano (Mount Doom) + Elf (Legolas)</p>
                <p>Better luck tomorrow!</p>
            </div>
            
            <div class="mt-6">
                <h3 class="font-bold mb-4">Previous Emoji Challenges</h3>
                <div class="space-y-4">
                    <div class="bg-gray-50 p-4 rounded-lg flex justify-between items-center">
                        <div>
                            <div class="text-2xl mb-1">🚢❄️💑💔</div>
                            <div class="text-sm font-medium">Titanic</div>
                        </div>
                        <div class="text-sm text-gray-500">62% guessed correctly</div>
                    </div>
                    <div class="bg-gray-50 p-4 rounded-lg flex justify-between items-center">
                        <div>
                            <div class="text-2xl mb-1">👦🏻🧙‍♂️⚡🧹</div>
                            <div class="text-sm font-medium">Harry Potter</div>
                        </div>
                        <div class="text-sm text-gray-500">87% guessed correctly</div>
                    </div>
                    <div class="bg-gray-50 p-4 rounded-lg flex justify-between items-center">
                        <div>
                            <div class="text-2xl mb-1">🦁👑🌍</div>
                            <div class="text-sm font-medium">The Lion King</div>
                        </div>
                        <div class="text-sm text-gray-500">91% guessed correctly</div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Reminder Modal -->
    <div 
        x-show="showReminderModal" 
        class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50"
        x-transition
    >
        <div class="bg-white rounded-lg p-6 max-w-md w-full">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-xl font-bold">Set Daily Reminder</h3>
                <button @click="showReminderModal = false" class="text-gray-500 hover:text-gray-700">
                    <i class="fas fa-times"></i>
                </button>
            </div>
            
            <div class="mb-4">
                <label class="block text-sm font-medium text-gray-700 mb-1">Reminder Type</label>
                <div class="grid grid-cols-2 gap-4">
                    <label class="flex items-center p-3 border rounded-lg cursor-pointer hover:bg-gray-50">
                        <input type="radio" name="reminderType" value="email" checked class="mr-2">
                        <span><i class="fas fa-envelope mr-1"></i> Email</span>
                    </label>
                    <label class="flex items-center p-3 border rounded-lg cursor-pointer hover:bg-gray-50">
                        <input type="radio" name="reminderType" value="discord" class="mr-2">
                        <span><i class="fab fa-discord mr-1"></i> Discord</span>
                    </label>
                </div>
            </div>
            
            <div class="mb-4">
                <label class="block text-sm font-medium text-gray-700 mb-1">Email Address</label>
                <input 
                    type="email" 
                    class="w-full p-2 border border-gray-300 rounded focus:outline-none focus:ring-2 focus:ring-indigo-500"
                    placeholder="your@email.com"
                />
            </div>
            
            <div class="mb-4">
                <label class="block text-sm font-medium text-gray-700 mb-1">Reminder Time</label>
                <select class="w-full p-2 border border-gray-300 rounded focus:outline-none focus:ring-2 focus:ring-indigo-500">
                    <option value="8">8:00 AM</option>
                    <option value="10">10:00 AM</option>
                    <option value="12">12:00 PM</option>
                    <option value="15">3:00 PM</option>
                    <option value="18">6:00 PM</option>
                    <option value="20">8:00 PM</option>
                </select>
            </div>
            
            <div class="flex justify-end">
                <button 
                    class="bg-indigo-600 hover:bg-indigo-700 text-white font-medium py-2 px-4 rounded-lg transition"
                    @click="setReminder()"
                >
                    Save Reminder
                </button>
            </div>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/confetti-js@0.0.18/dist/index.min.js"></script>
    <script>
        function app() {
            return {
                activeTab: 0,
                showReminderModal: false,
                showEmojiHint: false,
                microStory: '',
                emojiGuess: '',
                triviaCorrect: false,
                emojiCorrect: false,
                
                userStats: {
                    currentStreak: 7,
                    totalPoints: 845,
                    rank: "Expert",
                    completedToday: 0
                },
                
                challenges: [
                    { name: "Word Guess", completed: false },
                    { name: "Drawing Challenge", completed: false },
                    { name: "Trivia Daily", completed: false },
                    { name: "Micro Story", completed: false },
                    { name: "Emoji Translation", completed: false }
                ],
                
                wordGuess: {
                    currentAttempt: 0,
                    currentGuess: '',
                    attempts: [
                        { word: '', status: [] },
                        { word: '', status: [] },
                        { word: '', status: [] },
                        { word: '', status: [] },
                        { word: '', status: [] },
                        { word: '', status: [] }
                    ],
                    solution: 'CHALK'
                },
                
                init() {
                    this.initTriviaChart();
                    // Check local storage for completed challenges
                    const savedData = localStorage.getItem('dailyChallenges');
                    if (savedData) {
                        const data = JSON.parse(savedData);
                        this.challenges = data.challenges;
                        this.userStats.completedToday = data.completedToday;
                    }
                },
                
                setActiveTab(index) {
                    this.activeTab = index;
                },
                
                getCurrentDate() {
                    const now = new Date();
                    const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
                    return now.toLocaleDateString('en-US', options);
                },
                
                completeChallenge(index) {
                    if (!this.challenges[index].completed) {
                        this.challenges[index].completed = true;
                        this.userStats.completedToday++;
                        
                        // Show confetti for completed challenge
                        const confettiSettings = { target: 'confetti-canvas', max: 100, size: 2, animate: true };
                        const confetti = new ConfettiGenerator(confettiSettings);
                        confetti.render();
                        
                        setTimeout(() => {
                            confetti.clear();
                        }, 3000);
                        
                        // Save to localStorage
                        localStorage.setItem('dailyChallenges', JSON.stringify({
                            challenges: this.challenges,
                            completedToday: this.userStats.completedToday
                        }));
                    }
                },
                
                addLetter(letter) {
                    if (this.wordGuess.currentGuess.length < 5 && this.wordGuess.currentAttempt < 6) {
                        this.wordGuess.currentGuess += letter.toUpperCase();
                        this.wordGuess.attempts[this.wordGuess.currentAttempt].word = this.wordGuess.currentGuess;
                    }
                },
                
                deleteLetter() {
                    if (this.wordGuess.currentGuess.length > 0) {
                        this.wordGuess.currentGuess = this.wordGuess.currentGuess.slice(0, -1);
                        this.wordGuess.attempts[this.wordGuess.currentAttempt].word = this.wordGuess.currentGuess;
                    }
                },
                
                submitWord() {
                    if (this.wordGuess.currentGuess.length !== 5) return;
                    
                    const solution = this.wordGuess.solution;
                    const status = [];
                    
                    // Check each letter
                    for (let i = 0; i < 5; i++) {
                        const letter = this.wordGuess.currentGuess[i];
                        if (letter === solution[i]) {
                            status.push('correct');
                        } else if (solution.includes(letter)) {
                            status.push('present');
                        } else {
                            status.push('absent');
                        }
                    }
                    
                    this.wordGuess.attempts[this.wordGuess.currentAttempt].status = status;
                    
                    // Check if word is correct
                    if (this.wordGuess.currentGuess === solution) {
                        this.completeChallenge(0);
                        return;
                    }
                    
                    // Move to next attempt
                    this.wordGuess.currentAttempt++;
                    this.wordGuess.currentGuess = '';
                    
                    // Check if out of attempts
                    if (this.wordGuess.currentAttempt >= 6) {
                        alert("You've used all your attempts! The word was: " + solution);
                    }
                },
                
                checkTriviaAnswer(answer) {
                    if (answer === 'Venus') {
                        this.triviaCorrect = true;
                    } else {
                        this.triviaCorrect = false;
                    }
                    this.completeChallenge(2);
                },
                
                checkEmojiAnswer() {
                    const correctAnswer = "the lord of the rings";
                    const userAnswer = this.emojiGuess.trim().toLowerCase();
                    
                    if (userAnswer === correctAnswer) {
                        this.emojiCorrect = true;
                    } else {
                        this.emojiCorrect = false;
                    }
                    this.completeChallenge(4);
                },
                
                countWords(text) {
                    return text.trim().split(/\s+/).filter(word => word.length > 0).length;
                },
                
                setReminder() {
                    this.showReminderModal = false;
                    setTimeout(() => {
                        alert("Reminder set! You'll receive daily notifications to complete your challenges.");
                    }, 300);
                },
                
                initTriviaChart() {
                    setTimeout(() => {
                        const ctx = document.getElementById('triviaChart');
                        if (ctx) {
                            new Chart(ctx, {
                                type: 'pie',
                                data: {
                                    labels: ['Venus', 'Mars', 'Jupiter', 'Uranus'],
                                    datasets: [{
                                        data: [42, 30, 16, 12],
                                        backgroundColor: [
                                            '#10b981',
                                            '#ef4444',
                                            '#ef4444',
                                            '#ef4444'
                                        ]
                                    }]
                                },
                                options: {
                                    responsive: true,
                                    plugins: {
                                        legend: {
                                            position: 'bottom'
                                        },
                                        tooltip: {
                                            callbacks: {
                                                label: function(context) {
                                                    const label = context.label || '';
                                                    const value = context.parsed || 0;
                                                    const total = context.dataset.data.reduce((a, b) => a + b, 0);
                                                    const percentage = Math.round((value / total) * 100);
                                                    return `${label}: ${percentage}%`;
                                                }
                                            }
                                        }
                                    }
                                }
                            });
                        }
                    }, 100);
                }
            };
        }
    </script>
</body>
</html>
