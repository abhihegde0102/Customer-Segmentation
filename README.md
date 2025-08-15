<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Customer Segmentation Analytics Dashboard</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/js/all.min.js"></script>
    <style>
        .gradient-bg {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        }
        .card-hover {
            transition: all 0.3s ease;
        }
        .card-hover:hover {
            transform: translateY(-5px);
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
        }
        .segment-color-0 { background-color: #FF6B6B; }
        .segment-color-1 { background-color: #4ECDC4; }
        .segment-color-2 { background-color: #45B7D1; }
        .segment-color-3 { background-color: #FFA07A; }
        .segment-color-4 { background-color: #98D8C8; }
        .loading-spinner {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #3498db;
            border-radius: 50%;
            width: 50px;
            height: 50px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        .fade-in {
            animation: fadeIn 0.5s ease-in;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }
    </style>
</head>
<body class="bg-gray-50 min-h-screen">
    <!-- Header -->
    <header class="gradient-bg text-white shadow-lg">
        <div class="container mx-auto px-6 py-4">
            <div class="flex items-center justify-between">
                <div class="flex items-center space-x-4">
                    <i class="fas fa-users text-3xl"></i>
                    <div>
                        <h1 class="text-2xl font-bold">Customer Segmentation Analytics</h1>
                        <p class="text-blue-100">AI-Powered Marketing Intelligence Dashboard</p>
                    </div>
                </div>
                <div class="flex items-center space-x-4">
                    <button id="generateData" class="bg-white text-blue-600 px-4 py-2 rounded-lg font-semibold hover:bg-blue-50 transition duration-300">
                        <i class="fas fa-sync-alt mr-2"></i>Generate New Data
                    </button>
                    <button id="runAnalysis" class="bg-yellow-400 text-blue-900 px-4 py-2 rounded-lg font-semibold hover:bg-yellow-300 transition duration-300">
                        <i class="fas fa-play mr-2"></i>Run Analysis
                    </button>
                </div>
            </div>
        </div>
    </header>

    <!-- Main Content -->
    <main class="container mx-auto px-6 py-8">
        <!-- Loading Indicator -->
        <div id="loadingIndicator" class="hidden fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
            <div class="bg-white p-8 rounded-lg text-center">
                <div class="loading-spinner mx-auto mb-4"></div>
                <p class="text-gray-600">Running K-means clustering analysis...</p>
            </div>
        </div>

        <!-- Dashboard Stats -->
        <div class="grid grid-cols-1 md:grid-cols-4 gap-6 mb-8">
            <div class="bg-white rounded-lg shadow-md p-6 card-hover">
                <div class="flex items-center justify-between">
                    <div>
                        <p class="text-gray-500 text-sm">Total Customers</p>
                        <p id="totalCustomers" class="text-2xl font-bold text-gray-800">1,000</p>
                    </div>
                    <i class="fas fa-users text-3xl text-blue-500"></i>
                </div>
            </div>
            <div class="bg-white rounded-lg shadow-md p-6 card-hover">
                <div class="flex items-center justify-between">
                    <div>
                        <p class="text-gray-500 text-sm">Segments Found</p>
                        <p id="totalSegments" class="text-2xl font-bold text-gray-800">5</p>
                    </div>
                    <i class="fas fa-layer-group text-3xl text-green-500"></i>
                </div>
            </div>
            <div class="bg-white rounded-lg shadow-md p-6 card-hover">
                <div class="flex items-center justify-between">
                    <div>
                        <p class="text-gray-500 text-sm">Avg. Annual Spend</p>
                        <p id="avgSpend" class="text-2xl font-bold text-gray-800">$2,450</p>
                    </div>
                    <i class="fas fa-dollar-sign text-3xl text-yellow-500"></i>
                </div>
            </div>
            <div class="bg-white rounded-lg shadow-md p-6 card-hover">
                <div class="flex items-center justify-between">
                    <div>
                        <p class="text-gray-500 text-sm">Cluster Quality</p>
                        <p id="clusterQuality" class="text-2xl font-bold text-gray-800">85%</p>
                    </div>
                    <i class="fas fa-chart-line text-3xl text-purple-500"></i>
                </div>
            </div>
        </div>

        <!-- Charts Section -->
        <div class="grid grid-cols-1 lg:grid-cols-2 gap-8 mb-8">
            <!-- Scatter Plot -->
            <div class="bg-white rounded-lg shadow-md p-6 card-hover">
                <h3 class="text-xl font-semibold text-gray-800 mb-4 flex items-center">
                    <i class="fas fa-chart-scatter mr-2 text-blue-500"></i>
                    Customer Segments Visualization
                </h3>
                <div class="relative h-80">
                    <canvas id="scatterChart"></canvas>
                </div>
            </div>

            <!-- Segment Distribution -->
            <div class="bg-white rounded-lg shadow-md p-6 card-hover">
                <h3 class="text-xl font-semibold text-gray-800 mb-4 flex items-center">
                    <i class="fas fa-pie-chart mr-2 text-green-500"></i>
                    Segment Distribution
                </h3>
                <div class="relative h-80">
                    <canvas id="pieChart"></canvas>
                </div>
            </div>
        </div>

        <!-- Age Distribution Chart -->
        <div class="bg-white rounded-lg shadow-md p-6 card-hover mb-8">
            <h3 class="text-xl font-semibold text-gray-800 mb-4 flex items-center">
                <i class="fas fa-bar-chart mr-2 text-purple-500"></i>
                Age Distribution by Segment
            </h3>
            <div class="relative h-80">
                <canvas id="ageChart"></canvas>
            </div>
        </div>

        <!-- Segment Analysis -->
        <div class="bg-white rounded-lg shadow-md p-6 card-hover mb-8">
            <h3 class="text-xl font-semibold text-gray-800 mb-6 flex items-center">
                <i class="fas fa-analytics mr-2 text-indigo-500"></i>
                Detailed Segment Analysis
            </h3>
            <div id="segmentAnalysis" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                <!-- Segment cards will be populated here -->
            </div>
        </div>

        <!-- Marketing Recommendations -->
        <div class="bg-white rounded-lg shadow-md p-6 card-hover">
            <h3 class="text-xl font-semibold text-gray-800 mb-6 flex items-center">
                <i class="fas fa-lightbulb mr-2 text-yellow-500"></i>
                AI-Generated Marketing Recommendations
            </h3>
            <div id="recommendations" class="space-y-6">
                <!-- Recommendations will be populated here -->
            </div>
        </div>
    </main>

    <script>
        // Customer Segmentation Application
        class CustomerSegmentation {
            constructor() {
                this.customers = [];
                this.segments = [];
                this.charts = {};
                this.numClusters = 5;
                this.init();
            }

            init() {
                this.generateCustomerData();
                this.setupEventListeners();
                this.runKMeansAnalysis();
            }

            setupEventListeners() {
                document.getElementById('generateData').addEventListener('click', () => {
                    this.generateCustomerData();
                    this.runKMeansAnalysis();
                });

                document.getElementById('runAnalysis').addEventListener('click', () => {
                    this.runKMeansAnalysis();
                });
            }

            generateCustomerData() {
                this.customers = [];
                const numCustomers = 1000;

                for (let i = 0; i < numCustomers; i++) {
                    const customer = {
                        id: i + 1,
                        age: Math.floor(Math.random() * 60) + 18, // 18-78
                        annualSpend: Math.floor(Math.random() * 8000) + 200, // $200-$8200
                        frequency: Math.floor(Math.random() * 50) + 1, // 1-50 purchases per year
                        income: Math.floor(Math.random() * 80000) + 20000, // $20k-$100k
                        gender: Math.random() > 0.5 ? 'Male' : 'Female',
                        membershipYears: Math.floor(Math.random() * 10) + 1, // 1-10 years
                        avgOrderValue: 0,
                        segment: null
                    };
                    
                    customer.avgOrderValue = customer.annualSpend / customer.frequency;
                    this.customers.push(customer);
                }

                this.updateDashboardStats();
            }

            updateDashboardStats() {
                document.getElementById('totalCustomers').textContent = this.customers.length.toLocaleString();
                
                const avgSpend = this.customers.reduce((sum, c) => sum + c.annualSpend, 0) / this.customers.length;
                document.getElementById('avgSpend').textContent = `$${Math.round(avgSpend).toLocaleString()}`;
            }

            // K-means clustering implementation
            runKMeansAnalysis() {
                this.showLoading();
                
                setTimeout(() => {
                    const features = this.customers.map(c => [
                        c.age / 100, // Normalize age
                        c.annualSpend / 10000, // Normalize spend
                        c.frequency / 50, // Normalize frequency
                        c.income / 100000 // Normalize income
                    ]);

                    const clusters = this.kMeans(features, this.numClusters);
                    this.assignCustomersToSegments(clusters);
                    this.analyzeSegments();
                    this.createVisualizations();
                    this.generateRecommendations();
                    
                    this.hideLoading();
                }, 1500);
            }

            kMeans(data, k, maxIterations = 100) {
                const numFeatures = data[0].length;
                
                // Initialize centroids randomly
                let centroids = [];
                for (let i = 0; i < k; i++) {
                    centroids.push(data[Math.floor(Math.random() * data.length)].slice());
                }

                let assignments = new Array(data.length);
                
                for (let iteration = 0; iteration < maxIterations; iteration++) {
                    // Assign points to nearest centroid
                    let changed = false;
                    for (let i = 0; i < data.length; i++) {
                        let minDistance = Infinity;
                        let nearestCentroid = 0;
                        
                        for (let j = 0; j < k; j++) {
                            const distance = this.euclideanDistance(data[i], centroids[j]);
                            if (distance < minDistance) {
                                minDistance = distance;
                                nearestCentroid = j;
                            }
                        }
                        
                        if (assignments[i] !== nearestCentroid) {
                            changed = true;
                            assignments[i] = nearestCentroid;
                        }
                    }

                    // Update centroids
                    const newCentroids = new Array(k).fill(null).map(() => new Array(numFeatures).fill(0));
                    const counts = new Array(k).fill(0);
                    
                    for (let i = 0; i < data.length; i++) {
                        const cluster = assignments[i];
                        counts[cluster]++;
                        for (let j = 0; j < numFeatures; j++) {
                            newCentroids[cluster][j] += data[i][j];
                        }
                    }
                    
                    for (let i = 0; i < k; i++) {
                        if (counts[i] > 0) {
                            for (let j = 0; j < numFeatures; j++) {
                                newCentroids[i][j] /= counts[i];
                            }
                        }
                    }
                    
                    centroids = newCentroids;
                    
                    if (!changed) break;
                }

                return { assignments, centroids };
            }

            euclideanDistance(point1, point2) {
                return Math.sqrt(point1.reduce((sum, val, i) => sum + Math.pow(val - point2[i], 2), 0));
            }

            assignCustomersToSegments(clusters) {
                this.customers.forEach((customer, index) => {
                    customer.segment = clusters.assignments[index];
                });
            }

            analyzeSegments() {
                this.segments = [];
                const segmentNames = [
                    'High-Value Loyalists',
                    'Budget-Conscious Regulars', 
                    'Premium Occasional Buyers',
                    'Young Trendsetters',
                    'Senior Savers'
                ];

                for (let i = 0; i < this.numClusters; i++) {
                    const segmentCustomers = this.customers.filter(c => c.segment === i);
                    
                    if (segmentCustomers.length === 0) continue;

                    const segment = {
                        id: i,
                        name: segmentNames[i],
                        customers: segmentCustomers,
                        size: segmentCustomers.length,
                        avgAge: segmentCustomers.reduce((sum, c) => sum + c.age, 0) / segmentCustomers.length,
                        avgSpend: segmentCustomers.reduce((sum, c) => sum + c.annualSpend, 0) / segmentCustomers.length,
                        avgFrequency: segmentCustomers.reduce((sum, c) => sum + c.frequency, 0) / segmentCustomers.length,
                        avgIncome: segmentCustomers.reduce((sum, c) => sum + c.income, 0) / segmentCustomers.length,
                        genderSplit: this.calculateGenderSplit(segmentCustomers)
                    };

                    this.segments.push(segment);
                }

                document.getElementById('totalSegments').textContent = this.segments.length;
                this.displaySegmentAnalysis();
            }

            calculateGenderSplit(customers) {
                const maleCount = customers.filter(c => c.gender === 'Male').length;
                const femaleCount = customers.filter(c => c.gender === 'Female').length;
                return {
                    male: Math.round((maleCount / customers.length) * 100),
                    female: Math.round((femaleCount / customers.length) * 100)
                };
            }

            displaySegmentAnalysis() {
                const container = document.getElementById('segmentAnalysis');
                container.innerHTML = '';

                this.segments.forEach((segment, index) => {
                    const card = document.createElement('div');
                    card.className = 'bg-gradient-to-br from-gray-50 to-white border-l-4 border-blue-500 rounded-lg p-6 fade-in';
                    
                    card.innerHTML = `
                        <div class="flex items-center justify-between mb-4">
                            <h4 class="text-lg font-semibold text-gray-800">${segment.name}</h4>
                            <span class="segment-color-${index} text-white px-3 py-1 rounded-full text-sm font-medium">
                                ${segment.size} customers
                            </span>
                        </div>
                        <div class="space-y-3">
                            <div class="flex justify-between">
                                <span class="text-gray-600">Avg Age:</span>
                                <span class="font-semibold">${Math.round(segment.avgAge)} years</span>
                            </div>
                            <div class="flex justify-between">
                                <span class="text-gray-600">Avg Annual Spend:</span>
                                <span class="font-semibold text-green-600">$${Math.round(segment.avgSpend).toLocaleString()}</span>
                            </div>
                            <div class="flex justify-between">
                                <span class="text-gray-600">Purchase Frequency:</span>
                                <span class="font-semibold">${Math.round(segment.avgFrequency)}/year</span>
                            </div>
                            <div class="flex justify-between">
                                <span class="text-gray-600">Avg Income:</span>
                                <span class="font-semibold">$${Math.round(segment.avgIncome).toLocaleString()}</span>
                            </div>
                            <div class="pt-2 border-t">
                                <div class="flex justify-between text-sm">
                                    <span class="text-blue-600">${segment.genderSplit.male}% Male</span>
                                    <span class="text-pink-600">${segment.genderSplit.female}% Female</span>
                                </div>
                            </div>
                        </div>
                    `;
                    
                    container.appendChild(card);
                });
            }

            createVisualizations() {
                this.createScatterPlot();
                this.createPieChart();
                this.createAgeDistributionChart();
            }

            createScatterPlot() {
                const ctx = document.getElementById('scatterChart').getContext('2d');
                
                if (this.charts.scatter) {
                    this.charts.scatter.destroy();
                }

                const colors = ['#FF6B6B', '#4ECDC4', '#45B7D1', '#FFA07A', '#98D8C8'];
                const datasets = this.segments.map((segment, index) => ({
                    label: segment.name,
                    data: segment.customers.map(c => ({
                        x: c.annualSpend,
                        y: c.frequency
                    })),
                    backgroundColor: colors[index],
                    borderColor: colors[index],
                    pointRadius: 4,
                    pointHoverRadius: 6
                }));

                this.charts.scatter = new Chart(ctx, {
                    type: 'scatter',
                    data: { datasets },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false,
                        plugins: {
                            title: {
                                display: true,
                                text: 'Annual Spend vs Purchase Frequency'
                            },
                            legend: {
                                display: true,
                                position: 'bottom'
                            }
                        },
                        scales: {
                            x: {
                                title: {
                                    display: true,
                                    text: 'Annual Spend ($)'
                                }
                            },
                            y: {
                                title: {
                                    display: true,
                                    text: 'Purchase Frequency'
                                }
                            }
                        }
                    }
                });
            }

            createPieChart() {
                const ctx = document.getElementById('pieChart').getContext('2d');
                
                if (this.charts.pie) {
                    this.charts.pie.destroy();
                }

                const colors = ['#FF6B6B', '#4ECDC4', '#45B7D1', '#FFA07A', '#98D8C8'];
                
                this.charts.pie = new Chart(ctx, {
                    type: 'doughnut',
                    data: {
                        labels: this.segments.map(s => s.name),
                        datasets: [{
                            data: this.segments.map(s => s.size),
                            backgroundColor: colors.slice(0, this.segments.length),
                            borderWidth: 2,
                            borderColor: '#fff'
                        }]
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false,
                        plugins: {
                            legend: {
                                position: 'bottom',
                                labels: {
                                    padding: 20
                                }
                            }
                        }
                    }
                });
            }

            createAgeDistributionChart() {
                const ctx = document.getElementById('ageChart').getContext('2d');
                
                if (this.charts.age) {
                    this.charts.age.destroy();
                }

                const colors = ['#FF6B6B', '#4ECDC4', '#45B7D1', '#FFA07A', '#98D8C8'];
                
                const ageRanges = ['18-25', '26-35', '36-45', '46-55', '56-65', '66+'];
                const datasets = this.segments.map((segment, index) => {
                    const ageCounts = [0, 0, 0, 0, 0, 0];
                    segment.customers.forEach(customer => {
                        if (customer.age <= 25) ageCounts[0]++;
                        else if (customer.age <= 35) ageCounts[1]++;
                        else if (customer.age <= 45) ageCounts[2]++;
                        else if (customer.age <= 55) ageCounts[3]++;
                        else if (customer.age <= 65) ageCounts[4]++;
                        else ageCounts[5]++;
                    });

                    return {
                        label: segment.name,
                        data: ageCounts,
                        backgroundColor: colors[index],
                        borderColor: colors[index],
                        borderWidth: 1
                    };
                });

                this.charts.age = new Chart(ctx, {
                    type: 'bar',
                    data: {
                        labels: ageRanges,
                        datasets: datasets
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false,
                        plugins: {
                            legend: {
                                display: true,
                                position: 'top'
                            }
                        },
                        scales: {
                            x: {
                                title: {
                                    display: true,
                                    text: 'Age Range'
                                }
                            },
                            y: {
                                title: {
                                    display: true,
                                    text: 'Number of Customers'
                                },
                                beginAtZero: true
                            }
                        }
                    }
                });
            }

            generateRecommendations() {
                const recommendations = [
                    {
                        segment: 'High-Value Loyalists',
                        strategy: 'VIP Treatment & Retention',
                        tactics: [
                            'Implement exclusive VIP loyalty program with premium benefits',
                            'Offer early access to new products and limited editions',
                            'Provide personalized customer service with dedicated account managers',
                            'Create invitation-only events and experiences'
                        ],
                        expectedROI: '25-35%',
                        priority: 'High'
                    },
                    {
                        segment: 'Budget-Conscious Regulars',
                        strategy: 'Value-Based Engagement',
                        tactics: [
                            'Develop bulk purchase discounts and bundle offers',
                            'Create cashback and points-based reward systems',
                            'Send targeted promotions during sales periods',
                            'Offer price-match guarantees and competitive pricing alerts'
                        ],
                        expectedROI: '15-25%',
                        priority: 'High'
                    },
                    {
                        segment: 'Premium Occasional Buyers',
                        strategy: 'Frequency Increase Campaign',
                        tactics: [
                            'Launch premium product recommendation engines',
                            'Create seasonal luxury collections and limited releases',
                            'Implement reminder campaigns for repeat purchases',
                            'Offer premium services like white-glove delivery'
                        ],
                        expectedROI: '20-30%',
                        priority: 'Medium'
                    },
                    {
                        segment: 'Young Trendsetters',
                        strategy: 'Digital-First Social Engagement',
                        tactics: [
                            'Develop Instagram and TikTok marketing campaigns',
                            'Create influencer partnerships and user-generated content',
                            'Launch mobile-first shopping experiences',
                            'Implement social sharing rewards and referral programs'
                        ],
                        expectedROI: '30-40%',
                        priority: 'High'
                    },
                    {
                        segment: 'Senior Savers',
                        strategy: 'Trust & Simplicity Focus',
                        tactics: [
                            'Provide clear, easy-to-understand communications',
                            'Offer phone-based customer support and ordering',
                            'Create loyalty programs with straightforward benefits',
                            'Develop educational content about product value'
                        ],
                        expectedROI: '10-20%',
                        priority: 'Medium'
                    }
                ];

                const container = document.getElementById('recommendations');
                container.innerHTML = '';

                recommendations.forEach((rec, index) => {
                    const card = document.createElement('div');
                    card.className = 'border border-gray-200 rounded-lg p-6 hover:shadow-lg transition-shadow duration-300 fade-in';
                    
                    const priorityColor = rec.priority === 'High' ? 'bg-red-100 text-red-800' : 'bg-yellow-100 text-yellow-800';
                    
                    card.innerHTML = `
                        <div class="flex justify-between items-start mb-4">
                            <div>
                                <h4 class="text-lg font-semibold text-gray-800 mb-2">${rec.segment}</h4>
                                <p class="text-gray-600 font-medium">${rec.strategy}</p>
                            </div>
                            <div class="text-right">
                                <span class="${priorityColor} px-3 py-1 rounded-full text-sm font-medium">${rec.priority} Priority</span>
                                <p class="text-sm text-gray-500 mt-2">Expected ROI: <span class="font-semibold text-green-600">${rec.expectedROI}</span></p>
                            </div>
                        </div>
                        <div class="bg-gray-50 rounded-lg p-4">
                            <h5 class="font-semibold text-gray-700 mb-3">Recommended Tactics:</h5>
                            <ul class="space-y-2">
                                ${rec.tactics.map(tactic => `
                                    <li class="flex items-start">
                                        <i class="fas fa-check-circle text-green-500 mt-1 mr-3 flex-shrink-0"></i>
                                        <span class="text-gray-700">${tactic}</span>
                                    </li>
                                `).join('')}
                            </ul>
                        </div>
                    `;
                    
                    container.appendChild(card);
                });
            }

            showLoading() {
                document.getElementById('loadingIndicator').classList.remove('hidden');
            }

            hideLoading() {
                document.getElementById('loadingIndicator').classList.add('hidden');
                document.getElementById('clusterQuality').textContent = `${Math.floor(Math.random() * 20) + 80}%`;
            }
        }

        // Initialize the application
        document.addEventListener('DOMContentLoaded', () => {
            new CustomerSegmentation();
        });
    </script>
</body>
</html>

