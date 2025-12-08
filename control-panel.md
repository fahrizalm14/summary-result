```ts
import React, { useState } from 'react';
import { Server, MessageSquare, Clock, Cpu, HardDrive, Activity, Play, Square, RotateCw, Terminal, Key, Send, Image, QrCode, Calendar, CheckCircle, XCircle, AlertCircle, Globe, Zap, Bell } from 'lucide-react';

export default function Dashboard() {
  const [activeTab, setActiveTab] = useState('servers');
  const [servers, setServers] = useState([
    { id: 1, name: 'Production Server', ip: '192.168.1.100', status: 'online', cpu: 45, ram: 62, disk: 78, uptime: '15d 4h' }
  ]);
  const [waDevices, setWaDevices] = useState([
    { id: 1, name: 'WA Gateway 1', phone: '+62812345678', status: 'connected', queue: 5 }
  ]);
  const [cronJobs, setCronJobs] = useState([
    { id: 1, name: 'Backup Database', schedule: '0 2 * * *', enabled: true, lastRun: '2h ago', status: 'success' }
  ]);
  const [webapps, setWebapps] = useState([
    { id: 1, name: 'API Production', url: 'https://api.example.com/health', status: 'up', responseTime: 145, uptime: 99.9, interval: 60, lastCheck: '30s ago' },
    { id: 2, name: 'Dashboard Frontend', url: 'https://dashboard.example.com', status: 'up', responseTime: 89, uptime: 100, interval: 30, lastCheck: '15s ago' }
  ]);
  const [showQR, setShowQR] = useState(false);

  const StatusBadge = ({ status }) => {
    const colors = {
      online: 'bg-green-500',
      offline: 'bg-red-500',
      connected: 'bg-green-500',
      disconnected: 'bg-red-500',
      success: 'bg-green-500',
      failed: 'bg-red-500',
      pending: 'bg-yellow-500'
    };
    return <span className={`w-2 h-2 rounded-full ${colors[status] || 'bg-gray-500'}`} />;
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-slate-900 via-slate-800 to-slate-900 text-white p-6">
      <div className="max-w-7xl mx-auto">
        {/* Header */}
        <div className="mb-8">
          <h1 className="text-3xl font-bold bg-gradient-to-r from-blue-400 to-purple-500 bg-clip-text text-transparent">
            Control Panel
          </h1>
          <p className="text-slate-400 mt-1">Kelola server, WhatsApp, dan cron jobs dalam satu dashboard</p>
        </div>

        {/* Navigation Tabs */}
        <div className="flex gap-2 mb-6 bg-slate-800/50 p-1 rounded-lg backdrop-blur-sm">
          {[
            { id: 'servers', label: 'Server Management', icon: Server },
            { id: 'whatsapp', label: 'WhatsApp Gateway', icon: MessageSquare },
            { id: 'cronjobs', label: 'Cron Jobs', icon: Clock },
            { id: 'monitoring', label: 'WebApp Monitoring', icon: Globe }
          ].map(tab => (
            <button
              key={tab.id}
              onClick={() => setActiveTab(tab.id)}
              className={`flex items-center gap-2 px-4 py-2 rounded-md transition-all ${
                activeTab === tab.id
                  ? 'bg-blue-600 text-white shadow-lg'
                  : 'text-slate-400 hover:text-white hover:bg-slate-700/50'
              }`}
            >
              <tab.icon size={18} />
              <span className="font-medium">{tab.label}</span>
            </button>
          ))}
        </div>

        {/* Content Area */}
        <div className="space-y-6">
          {/* SERVER MANAGEMENT */}
          {activeTab === 'servers' && (
            <>
              <div className="bg-slate-800/50 backdrop-blur-sm rounded-xl p-6 border border-slate-700">
                <div className="flex justify-between items-center mb-4">
                  <h2 className="text-xl font-semibold">Servers</h2>
                  <button className="px-4 py-2 bg-blue-600 hover:bg-blue-700 rounded-lg transition-colors">
                    + Tambah Server
                  </button>
                </div>
                
                {servers.map(server => (
                  <div key={server.id} className="bg-slate-700/30 rounded-lg p-4 mb-4">
                    <div className="flex items-center justify-between mb-3">
                      <div className="flex items-center gap-3">
                        <StatusBadge status={server.status} />
                        <div>
                          <h3 className="font-semibold">{server.name}</h3>
                          <p className="text-sm text-slate-400">{server.ip}</p>
                        </div>
                      </div>
                      <div className="flex gap-2">
                        <button className="p-2 bg-green-600/20 hover:bg-green-600/30 rounded-lg transition-colors" title="Start">
                          <Play size={16} className="text-green-400" />
                        </button>
                        <button className="p-2 bg-red-600/20 hover:bg-red-600/30 rounded-lg transition-colors" title="Stop">
                          <Square size={16} className="text-red-400" />
                        </button>
                        <button className="p-2 bg-blue-600/20 hover:bg-blue-600/30 rounded-lg transition-colors" title="Restart">
                          <RotateCw size={16} className="text-blue-400" />
                        </button>
                        <button className="p-2 bg-slate-600/20 hover:bg-slate-600/30 rounded-lg transition-colors" title="Terminal">
                          <Terminal size={16} className="text-slate-400" />
                        </button>
                      </div>
                    </div>
                    
                    <div className="grid grid-cols-4 gap-4">
                      <div className="bg-slate-800/50 rounded-lg p-3">
                        <div className="flex items-center gap-2 text-blue-400 mb-1">
                          <Cpu size={16} />
                          <span className="text-xs">CPU</span>
                        </div>
                        <div className="text-xl font-bold">{server.cpu}%</div>
                      </div>
                      <div className="bg-slate-800/50 rounded-lg p-3">
                        <div className="flex items-center gap-2 text-purple-400 mb-1">
                          <Activity size={16} />
                          <span className="text-xs">RAM</span>
                        </div>
                        <div className="text-xl font-bold">{server.ram}%</div>
                      </div>
                      <div className="bg-slate-800/50 rounded-lg p-3">
                        <div className="flex items-center gap-2 text-orange-400 mb-1">
                          <HardDrive size={16} />
                          <span className="text-xs">Disk</span>
                        </div>
                        <div className="text-xl font-bold">{server.disk}%</div>
                      </div>
                      <div className="bg-slate-800/50 rounded-lg p-3">
                        <div className="flex items-center gap-2 text-green-400 mb-1">
                          <Clock size={16} />
                          <span className="text-xs">Uptime</span>
                        </div>
                        <div className="text-lg font-bold">{server.uptime}</div>
                      </div>
                    </div>
                  </div>
                ))}
              </div>

              <div className="bg-slate-800/50 backdrop-blur-sm rounded-xl p-6 border border-slate-700">
                <h2 className="text-xl font-semibold mb-4">Log Aktivitas</h2>
                <div className="space-y-2 text-sm">
                  <div className="flex items-center gap-3 p-2 bg-slate-700/30 rounded">
                    <CheckCircle size={16} className="text-green-400" />
                    <span className="text-slate-300">Service nginx restarted</span>
                    <span className="ml-auto text-slate-500">2m ago</span>
                  </div>
                  <div className="flex items-center gap-3 p-2 bg-slate-700/30 rounded">
                    <AlertCircle size={16} className="text-yellow-400" />
                    <span className="text-slate-300">High CPU usage detected</span>
                    <span className="ml-auto text-slate-500">15m ago</span>
                  </div>
                  <div className="flex items-center gap-3 p-2 bg-slate-700/30 rounded">
                    <CheckCircle size={16} className="text-green-400" />
                    <span className="text-slate-300">Command executed: systemctl status</span>
                    <span className="ml-auto text-slate-500">1h ago</span>
                  </div>
                </div>
              </div>
            </>
          )}

          {/* WHATSAPP GATEWAY */}
          {activeTab === 'whatsapp' && (
            <>
              <div className="bg-slate-800/50 backdrop-blur-sm rounded-xl p-6 border border-slate-700">
                <div className="flex justify-between items-center mb-4">
                  <h2 className="text-xl font-semibold">WhatsApp Devices</h2>
                  <button 
                    onClick={() => setShowQR(!showQR)}
                    className="px-4 py-2 bg-green-600 hover:bg-green-700 rounded-lg transition-colors flex items-center gap-2"
                  >
                    <QrCode size={18} />
                    Connect Device
                  </button>
                </div>

                {showQR && (
                  <div className="bg-slate-700/50 rounded-lg p-6 mb-4 text-center">
                    <div className="bg-white w-48 h-48 mx-auto rounded-lg flex items-center justify-center mb-3">
                      <QrCode size={64} className="text-slate-800" />
                    </div>
                    <p className="text-slate-400">Scan QR code dengan WhatsApp</p>
                  </div>
                )}

                {waDevices.map(device => (
                  <div key={device.id} className="bg-slate-700/30 rounded-lg p-4 mb-4">
                    <div className="flex items-center justify-between mb-3">
                      <div className="flex items-center gap-3">
                        <StatusBadge status={device.status} />
                        <div>
                          <h3 className="font-semibold">{device.name}</h3>
                          <p className="text-sm text-slate-400">{device.phone}</p>
                        </div>
                      </div>
                      <div className="flex items-center gap-4">
                        <div className="bg-blue-600/20 px-3 py-1 rounded-full">
                          <span className="text-blue-400 text-sm">Queue: {device.queue}</span>
                        </div>
                        <button className="px-3 py-1 bg-red-600/20 hover:bg-red-600/30 rounded-lg transition-colors text-red-400 text-sm">
                          Disconnect
                        </button>
                      </div>
                    </div>
                  </div>
                ))}
              </div>

              <div className="grid grid-cols-2 gap-6">
                <div className="bg-slate-800/50 backdrop-blur-sm rounded-xl p-6 border border-slate-700">
                  <h2 className="text-xl font-semibold mb-4 flex items-center gap-2">
                    <Send size={20} />
                    Kirim Pesan
                  </h2>
                  <div className="space-y-3">
                    <input 
                      type="text" 
                      placeholder="Nomor tujuan" 
                      className="w-full bg-slate-700/50 border border-slate-600 rounded-lg px-4 py-2 focus:outline-none focus:border-blue-500"
                    />
                    <textarea 
                      placeholder="Pesan..." 
                      rows="3"
                      className="w-full bg-slate-700/50 border border-slate-600 rounded-lg px-4 py-2 focus:outline-none focus:border-blue-500"
                    />
                    <div className="flex gap-2">
                      <button className="flex-1 px-4 py-2 bg-blue-600 hover:bg-blue-700 rounded-lg transition-colors">
                        Kirim Teks
                      </button>
                      <button className="px-4 py-2 bg-purple-600 hover:bg-purple-700 rounded-lg transition-colors">
                        <Image size={18} />
                      </button>
                    </div>
                  </div>
                </div>

                <div className="bg-slate-800/50 backdrop-blur-sm rounded-xl p-6 border border-slate-700">
                  <h2 className="text-xl font-semibold mb-4 flex items-center gap-2">
                    <Key size={20} />
                    API Configuration
                  </h2>
                  <div className="space-y-3">
                    <div>
                      <label className="text-sm text-slate-400 block mb-1">API Key</label>
                      <div className="bg-slate-700/50 border border-slate-600 rounded-lg px-4 py-2 font-mono text-sm">
                        sk_test_4eC39HqLyjWDarjtT1zdp7dc
                      </div>
                    </div>
                    <div>
                      <label className="text-sm text-slate-400 block mb-1">Webhook URL</label>
                      <input 
                        type="text" 
                        placeholder="https://your-domain.com/webhook" 
                        className="w-full bg-slate-700/50 border border-slate-600 rounded-lg px-4 py-2 focus:outline-none focus:border-blue-500"
                      />
                    </div>
                    <button className="w-full px-4 py-2 bg-slate-700 hover:bg-slate-600 rounded-lg transition-colors">
                      Generate New Key
                    </button>
                  </div>
                </div>
              </div>

              <div className="bg-slate-800/50 backdrop-blur-sm rounded-xl p-6 border border-slate-700">
                <h2 className="text-xl font-semibold mb-4">Message Logs</h2>
                <div className="space-y-2 text-sm">
                  <div className="flex items-center gap-3 p-2 bg-slate-700/30 rounded">
                    <CheckCircle size={16} className="text-green-400" />
                    <span className="text-slate-300">Pesan terkirim ke +6281234567890</span>
                    <span className="ml-auto text-slate-500">1m ago</span>
                  </div>
                  <div className="flex items-center gap-3 p-2 bg-slate-700/30 rounded">
                    <CheckCircle size={16} className="text-green-400" />
                    <span className="text-slate-300">Media terkirim ke +6281234567891</span>
                    <span className="ml-auto text-slate-500">5m ago</span>
                  </div>
                  <div className="flex items-center gap-3 p-2 bg-slate-700/30 rounded">
                    <XCircle size={16} className="text-red-400" />
                    <span className="text-slate-300">Gagal kirim ke +6281234567892</span>
                    <span className="ml-auto text-slate-500">10m ago</span>
                  </div>
                </div>
              </div>
            </>
          )}

          {/* CRON JOBS */}
          {activeTab === 'cronjobs' && (
            <>
              <div className="bg-slate-800/50 backdrop-blur-sm rounded-xl p-6 border border-slate-700">
                <div className="flex justify-between items-center mb-4">
                  <h2 className="text-xl font-semibold">Scheduled Jobs</h2>
                  <button className="px-4 py-2 bg-blue-600 hover:bg-blue-700 rounded-lg transition-colors">
                    + Buat Job Baru
                  </button>
                </div>

                {cronJobs.map(job => (
                  <div key={job.id} className="bg-slate-700/30 rounded-lg p-4 mb-4">
                    <div className="flex items-center justify-between mb-3">
                      <div className="flex items-center gap-3 flex-1">
                        <StatusBadge status={job.status} />
                        <div className="flex-1">
                          <h3 className="font-semibold">{job.name}</h3>
                          <div className="flex items-center gap-4 mt-1">
                            <span className="text-sm text-slate-400 font-mono">{job.schedule}</span>
                            <span className="text-xs text-slate-500">Last run: {job.lastRun}</span>
                          </div>
                        </div>
                      </div>
                      <div className="flex items-center gap-3">
                        <label className="relative inline-block w-12 h-6">
                          <input 
                            type="checkbox" 
                            checked={job.enabled}
                            onChange={() => {
                              setCronJobs(cronJobs.map(j => 
                                j.id === job.id ? {...j, enabled: !j.enabled} : j
                              ));
                            }}
                            className="sr-only peer"
                          />
                          <div className="w-12 h-6 bg-slate-600 rounded-full peer peer-checked:bg-green-600 transition-colors"></div>
                          <div className="absolute left-1 top-1 w-4 h-4 bg-white rounded-full transition-transform peer-checked:translate-x-6"></div>
                        </label>
                        <button className="p-2 bg-blue-600/20 hover:bg-blue-600/30 rounded-lg transition-colors" title="Run Now">
                          <Play size={16} className="text-blue-400" />
                        </button>
                        <button className="p-2 bg-slate-600/20 hover:bg-slate-600/30 rounded-lg transition-colors" title="Edit">
                          <Calendar size={16} className="text-slate-400" />
                        </button>
                      </div>
                    </div>
                  </div>
                ))}
              </div>

              <div className="bg-slate-800/50 backdrop-blur-sm rounded-xl p-6 border border-slate-700">
                <h2 className="text-xl font-semibold mb-4">Execution Logs</h2>
                <div className="space-y-2 text-sm">
                  <div className="flex items-center gap-3 p-2 bg-slate-700/30 rounded">
                    <CheckCircle size={16} className="text-green-400" />
                    <span className="text-slate-300">Backup Database completed</span>
                    <span className="ml-auto text-slate-500">2h ago</span>
                  </div>
                  <div className="flex items-center gap-3 p-2 bg-slate-700/30 rounded">
                    <CheckCircle size={16} className="text-green-400" />
                    <span className="text-slate-300">Clean temp files completed</span>
                    <span className="ml-auto text-slate-500">4h ago</span>
                  </div>
                  <div className="flex items-center gap-3 p-2 bg-slate-700/30 rounded">
                    <XCircle size={16} className="text-red-400" />
                    <span className="text-slate-300">Send report failed</span>
                    <span className="ml-auto text-slate-500">6h ago</span>
                  </div>
                </div>
              </div>
            </>
          )}

          {/* WEBAPP MONITORING */}
          {activeTab === 'monitoring' && (
            <>
              <div className="bg-slate-800/50 backdrop-blur-sm rounded-xl p-6 border border-slate-700">
                <div className="flex justify-between items-center mb-4">
                  <h2 className="text-xl font-semibold">WebApp Monitoring</h2>
                  <button className="px-4 py-2 bg-blue-600 hover:bg-blue-700 rounded-lg transition-colors">
                    + Monitor WebApp Baru
                  </button>
                </div>

                {webapps.map(app => (
                  <div key={app.id} className="bg-slate-700/30 rounded-lg p-4 mb-4">
                    <div className="flex items-center justify-between mb-3">
                      <div className="flex items-center gap-3 flex-1">
                        <StatusBadge status={app.status === 'up' ? 'online' : 'offline'} />
                        <div className="flex-1">
                          <h3 className="font-semibold">{app.name}</h3>
                          <p className="text-sm text-slate-400 break-all">{app.url}</p>
                        </div>
                      </div>
                      <div className="flex items-center gap-3">
                        <div className="bg-blue-600/20 px-3 py-1 rounded-full">
                          <span className="text-blue-400 text-sm">Every {app.interval}s</span>
                        </div>
                        <button className="p-2 bg-slate-600/20 hover:bg-slate-600/30 rounded-lg transition-colors" title="Edit">
                          <Bell size={16} className="text-slate-400" />
                        </button>
                      </div>
                    </div>
                    
                    <div className="grid grid-cols-4 gap-4">
                      <div className="bg-slate-800/50 rounded-lg p-3">
                        <div className="flex items-center gap-2 text-green-400 mb-1">
                          <CheckCircle size={16} />
                          <span className="text-xs">Status</span>
                        </div>
                        <div className="text-xl font-bold uppercase">{app.status}</div>
                      </div>
                      <div className="bg-slate-800/50 rounded-lg p-3">
                        <div className="flex items-center gap-2 text-purple-400 mb-1">
                          <Zap size={16} />
                          <span className="text-xs">Response Time</span>
                        </div>
                        <div className="text-xl font-bold">{app.responseTime}ms</div>
                      </div>
                      <div className="bg-slate-800/50 rounded-lg p-3">
                        <div className="flex items-center gap-2 text-blue-400 mb-1">
                          <Activity size={16} />
                          <span className="text-xs">Uptime</span>
                        </div>
                        <div className="text-xl font-bold">{app.uptime}%</div>
                      </div>
                      <div className="bg-slate-800/50 rounded-lg p-3">
                        <div className="flex items-center gap-2 text-orange-400 mb-1">
                          <Clock size={16} />
                          <span className="text-xs">Last Check</span>
                        </div>
                        <div className="text-lg font-bold">{app.lastCheck}</div>
                      </div>
                    </div>
                  </div>
                ))}
              </div>

              <div className="grid grid-cols-2 gap-6">
                <div className="bg-slate-800/50 backdrop-blur-sm rounded-xl p-6 border border-slate-700">
                  <h2 className="text-xl font-semibold mb-4">Response Time Chart</h2>
                  <div className="h-48 flex items-end justify-around gap-2">
                    {[120, 145, 98, 156, 89, 134, 145, 123, 167, 145].map((height, i) => (
                      <div key={i} className="flex-1 bg-gradient-to-t from-blue-600 to-purple-500 rounded-t" style={{height: `${(height/200)*100}%`}}></div>
                    ))}
                  </div>
                  <div className="flex justify-between text-xs text-slate-500 mt-2">
                    <span>10m ago</span>
                    <span>Now</span>
                  </div>
                </div>

                <div className="bg-slate-800/50 backdrop-blur-sm rounded-xl p-6 border border-slate-700">
                  <h2 className="text-xl font-semibold mb-4 flex items-center gap-2">
                    <Bell size={20} />
                    Notification Settings
                  </h2>
                  <div className="space-y-3">
                    <div>
                      <label className="text-sm text-slate-400 block mb-1">Alert When Down (minutes)</label>
                      <input 
                        type="number" 
                        defaultValue="5"
                        className="w-full bg-slate-700/50 border border-slate-600 rounded-lg px-4 py-2 focus:outline-none focus:border-blue-500"
                      />
                    </div>
                    <div>
                      <label className="text-sm text-slate-400 block mb-1">Email Notifications</label>
                      <input 
                        type="email" 
                        placeholder="admin@example.com"
                        className="w-full bg-slate-700/50 border border-slate-600 rounded-lg px-4 py-2 focus:outline-none focus:border-blue-500"
                      />
                    </div>
                    <div>
                      <label className="text-sm text-slate-400 block mb-1">Webhook URL</label>
                      <input 
                        type="text" 
                        placeholder="https://hooks.slack.com/..."
                        className="w-full bg-slate-700/50 border border-slate-600 rounded-lg px-4 py-2 focus:outline-none focus:border-blue-500"
                      />
                    </div>
                    <button className="w-full px-4 py-2 bg-blue-600 hover:bg-blue-700 rounded-lg transition-colors">
                      Save Settings
                    </button>
                  </div>
                </div>
              </div>

              <div className="bg-slate-800/50 backdrop-blur-sm rounded-xl p-6 border border-slate-700">
                <h2 className="text-xl font-semibold mb-4">Downtime History</h2>
                <div className="space-y-2 text-sm">
                  <div className="flex items-center gap-3 p-2 bg-slate-700/30 rounded">
                    <XCircle size={16} className="text-red-400" />
                    <span className="text-slate-300">API Production was down for 3m 24s</span>
                    <span className="ml-auto text-slate-500">2d ago</span>
                  </div>
                  <div className="flex items-center gap-3 p-2 bg-slate-700/30 rounded">
                    <AlertCircle size={16} className="text-yellow-400" />
                    <span className="text-slate-300">Dashboard Frontend slow response (2.5s)</span>
                    <span className="ml-auto text-slate-500">3d ago</span>
                  </div>
                  <div className="flex items-center gap-3 p-2 bg-slate-700/30 rounded">
                    <XCircle size={16} className="text-red-400" />
                    <span className="text-slate-300">API Production was down for 15m 12s</span>
                    <span className="ml-auto text-slate-500">1w ago</span>
                  </div>
                  <div className="flex items-center gap-3 p-2 bg-slate-700/30 rounded">
                    <CheckCircle size={16} className="text-green-400" />
                    <span className="text-slate-300">All services healthy - 30 days uptime</span>
                    <span className="ml-auto text-slate-500">1m ago</span>
                  </div>
                </div>
              </div>
            </>
          )}
        </div>
      </div>
    </div>
  );
}
```
