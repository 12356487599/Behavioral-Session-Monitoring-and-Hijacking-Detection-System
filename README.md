import { useState, useEffect } from 'react'
import { Button } from "/components/ui/button"
import { Card, CardHeader, CardTitle, CardContent, CardFooter } from "/components/ui/card"
import { Alert, AlertDescription, AlertTitle } from "/components/ui/alert"
import { Bell, Shield, X } from "lucide-react"

export default function SessionHijackingDefense() {
const \[activityData, setActivityData] = useState<{
mouseMovements: number\[]
clickTimestamps: number\[]
keyPressTimestamps: number\[]
lastActivity: number
}>({
mouseMovements: \[],
clickTimestamps: \[],
keyPressTimestamps: \[],
lastActivity: Date.now()
})

const \[alerts, setAlerts] = useState<{
id: string
type: 'mouse' | 'click' | 'keyboard' | 'inactivity'
message: string
timestamp: number
}\[]>(\[])

const \[showTerminateModal, setShowTerminateModal] = useState(false)

// Track mouse movements
useEffect(() => {
const handleMouseMove = () => {
const now = Date.now()
setActivityData(prev => ({
...prev,
mouseMovements: \[...prev.mouseMovements.slice(-99), now],
lastActivity: now
}))
}

```
window.addEventListener('mousemove', handleMouseMove)
return () => window.removeEventListener('mousemove', handleMouseMove)
```

}, \[])

// Track clicks
useEffect(() => {
const handleClick = () => {
const now = Date.now()
setActivityData(prev => ({
...prev,
clickTimestamps: \[...prev.clickTimestamps.slice(-99), now],
lastActivity: now
}))
}

```
window.addEventListener('click', handleClick)
return () => window.removeEventListener('click', handleClick)
```

}, \[])

// Track keyboard activity
useEffect(() => {
const handleKeyPress = () => {
const now = Date.now()
setActivityData(prev => ({
...prev,
keyPressTimestamps: \[...prev.keyPressTimestamps.slice(-99), now],
lastActivity: now
}))
}

```
window.addEventListener('keydown', handleKeyPress)
return () => window.removeEventListener('keydown', handleKeyPress)
```

}, \[])

// Detect anomalies
useEffect(() => {
const checkForAnomalies = () => {
const now = Date.now()
const { mouseMovements, clickTimestamps, keyPressTimestamps, lastActivity } = activityData

```
  // Check for rapid mouse movements (potential automated behavior)
  if (mouseMovements.length > 10) {
    const recentMovements = mouseMovements.slice(-10)
    const avgInterval = recentMovements.reduce((acc, curr, i, arr) => {
      if (i === 0) return acc
      return acc + (curr - arr[i - 1])
    }, 0) / (recentMovements.length - 1)

    if (avgInterval < 50) { // Less than 50ms between movements
      const alertExists = alerts.some(a => a.type === 'mouse')
      if (!alertExists) {
        setAlerts(prev => [...prev, {
          id: `mouse-${now}`,
          type: 'mouse',
          message: 'Unusually rapid mouse movements detected',
          timestamp: now
        }])
      }
    }
  }

  // Check for rapid clicks (potential automated behavior)
  if (clickTimestamps.length > 5) {
    const recentClicks = clickTimestamps.slice(-5)
    const avgClickInterval = recentClicks.reduce((acc, curr, i, arr) => {
      if (i === 0) return acc
      return acc + (curr - arr[i - 1])
    }, 0) / (recentClicks.length - 1)

    if (avgClickInterval < 200) { // Less than 200ms between clicks
      const alertExists = alerts.some(a => a.type === 'click')
      if (!alertExists) {
        setAlerts(prev => [...prev, {
          id: `click-${now}`,
          type: 'click',
          message: 'Unusually rapid clicking detected',
          timestamp: now
        }])
      }
    }
  }

  // Check for inactivity (potential session takeover)
  if (now - lastActivity > 300000) { // 5 minutes of inactivity
    const alertExists = alerts.some(a => a.type === 'inactivity')
    if (!alertExists) {
      setAlerts(prev => [...prev, {
        id: `inactivity-${now}`,
        type: 'inactivity',
        message: 'Extended period of inactivity detected',
        timestamp: now
      }])
    }
  }
}

const interval = setInterval(checkForAnomalies, 5000)
return () => clearInterval(interval)
```

}, \[activityData, alerts])

const dismissAlert = (id: string) => {
setAlerts(prev => prev.filter(alert => alert.id !== id))
}

const terminateSession = () => {
// In a real app, this would call an API to invalidate the session
alert('Session terminated successfully')
setShowTerminateModal(false)
}

return ( <Card className="w-full max-w-2xl mx-auto"> <CardHeader> <div className="flex items-center justify-between"> <CardTitle className="flex items-center gap-2"> <Shield className="w-5 h-5" />
Session Security Monitor </CardTitle> <div className="flex items-center gap-2"> <span className="text-sm text-gray-500">
Last activity: {new Date(activityData.lastActivity).toLocaleTimeString()} </span> </div> </div> </CardHeader>

```
  <CardContent className="space-y-4">
    <div className="grid grid-cols-3 gap-4">
      <div className="p-4 border rounded-lg">
        <h3 className="font-medium">Mouse Activity</h3>
        <p className="text-2xl font-bold">{activityData.mouseMovements.length}</p>
        <p className="text-sm text-gray-500">movements tracked</p>
      </div>
      <div className="p-4 border rounded-lg">
        <h3 className="font-medium">Click Activity</h3>
        <p className="text-2xl font-bold">{activityData.clickTimestamps.length}</p>
        <p className="text-sm text-gray-500">clicks tracked</p>
      </div>
      <div className="p-4 border rounded-lg">
        <h3 className="font-medium">Keyboard Activity</h3>
        <p className="text-2xl font-bold">{activityData.keyPressTimestamps.length}</p>
        <p className="text-sm text-gray-500">key presses</p>
      </div>
    </div>

    <div className="space-y-2">
      <h3 className="font-medium">Security Alerts</h3>
      {alerts.length === 0 ? (
        <div className="p-4 text-center text-gray-500 border rounded-lg">
          No security alerts detected
        </div>
      ) : (
        <div className="space-y-2">
          {alerts.map(alert => (
            <Alert key={alert.id} variant={alert.type === 'inactivity' ? 'default' : 'destructive'}>
              <Bell className="w-4 h-4" />
              <AlertTitle>{alert.type === 'inactivity' ? 'Inactivity Warning' : 'Suspicious Activity'}</AlertTitle>
              <AlertDescription className="flex justify-between items-center">
                <span>{alert.message}</span>
                <Button
                  variant="ghost"
                  size="sm"
                  onClick={() => dismissAlert(alert.id)}
                >
                  <X className="w-4 h-4" />
                </Button>
              </AlertDescription>
            </Alert>
          ))}
        </div>
      )}
    </div>
  </CardContent>

  <CardFooter className="flex justify-end">
    <Button
      variant="destructive"
      onClick={() => setShowTerminateModal(true)}
    >
      Terminate Session
    </Button>
  </CardFooter>

  {showTerminateModal && (
    <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4">
      <Card className="w-full max-w-md">
        <CardHeader>
          <CardTitle>Terminate Session?</CardTitle>
        </CardHeader>
        <CardContent>
          <p>Are you sure you want to terminate this session? You will need to log in again.</p>
        </CardContent>
        <CardFooter className="flex justify-end gap-2">
          <Button variant="outline" onClick={() => setShowTerminateModal(false)}>
            Cancel
          </Button>
          <Button variant="destructive" onClick={terminateSession}>
            Terminate
          </Button>
        </CardFooter>
      </Card>
    </div>
  )}
</Card>
```

)
}

Version 1 of 1
