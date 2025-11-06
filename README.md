local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local playerGui = game.Players.LocalPlayer:WaitForChild("PlayerGui")

local screenGui = Instance.new("ScreenGui", playerGui)

local function createDraggable(gui)
 local dragging, startPos, startInput
 gui.InputBegan:Connect(function(input)
  if input.UserInputType == Enum.UserInputType.MouseButton1 then
   dragging = true
   startPos = gui.Position
   startInput = input.Position
   input.Changed:Connect(function()
    if input.UserInputState == Enum.UserInputState.End then
     dragging = false
    end
   end)
  end
 end)
 gui.InputChanged:Connect(function(input)
  if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
   local delta = input.Position - startInput
   gui.Position = UDim2.new(
    startPos.X.Scale, startPos.X.Offset + delta.X,
    startPos.Y.Scale, startPos.Y.Offset + delta.Y
   )
  end
 end)
end

local container = Instance.new("Frame", screenGui)
container.Size = UDim2.new(0, 160, 0, 160)
container.Position = UDim2.new(0.5, -80, 0.5, -80)
container.AnchorPoint = Vector2.new(0.5, 0.5)
container.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
container.BackgroundTransparency = 0.3
container.BorderSizePixel = 0
Instance.new("UICorner", container).CornerRadius = UDim.new(0, 25)

local toggleBtn = Instance.new("TextButton", screenGui)
toggleBtn.Size = UDim2.new(0, 120, 0, 30)
toggleBtn.Position = UDim2.new(0.5, -60, 0.5, 90)
toggleBtn.AnchorPoint = Vector2.new(0.5, 0)
toggleBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
toggleBtn.TextColor3 = Color3.new(1,1,1)
toggleBtn.BorderSizePixel = 0
toggleBtn.Text = "Скрыть поле"
Instance.new("UICorner", toggleBtn).CornerRadius = UDim.new(0, 10)

toggleBtn.MouseButton1Click:Connect(function()
 container.Visible = not container.Visible
 toggleBtn.Text = container.Visible and "Скрыть поле" or "Показать поле"
end)

createDraggable(container)
createDraggable(toggleBtn)

local ball = Instance.new("Frame", container)
ball.Size = UDim2.new(0, 50, 0, 50)
ball.Position = UDim2.new(0.5, 0, 0.8, 0)
ball.AnchorPoint = Vector2.new(0.5, 0.5)
ball.BackgroundColor3 = Color3.new(1, 1, 1)
ball.BorderSizePixel = 0
Instance.new("UICorner", ball).CornerRadius = UDim.new(1, 0)

local gradient = Instance.new("UIGradient", ball)
gradient.Color = ColorSequence.new{
 ColorSequenceKeypoint.new(0, Color3.fromRGB(255,150,0)),
 ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255,255,100)),
 ColorSequenceKeypoint.new(1, Color3.fromRGB(255,50,0)),
}
gradient.Rotation = 45

local pulseTween = TweenService:Create(ball, TweenInfo.new(1, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), {Size=UDim2.new(0,60,0,60)})
pulseTween:Play()

local rotation = 0
RunService.Heartbeat:Connect(function(dt)
 if container.Visible then
  rotation = (rotation + 90 * dt) % 360
  gradient.Rotation = rotation
 end
end)

local gravity = 1000
local velocity = 0
local bounce = -0.6
local direction = 1
local speed = 200
local onGround = true
local jumpForce = -600
local jumpTimer = 0

RunService.Heartbeat:Connect(function(dt)
 if not container.Visible then return end

 local x = ball.Position.X.Offset + direction * speed * dt
 if x < ball.AbsoluteSize.X/2 then direction = 1 x = ball.AbsoluteSize.X/2
 elseif x > container.AbsoluteSize.X - ball.AbsoluteSize.X/2 then direction = -1 x = container.AbsoluteSize.X - ball.AbsoluteSize.X/2 end

 if onGround then
  jumpTimer = jumpTimer - dt
  if jumpTimer <= 0 then
   velocity = jumpForce
   onGround = false
   jumpTimer = math.random(2,4)
  end
 else
  velocity = velocity + gravity * dt
 end

 local y = ball.Position.Y.Offset + velocity * dt
 local floorY = container.AbsoluteSize.Y - ball.AbsoluteSize.Y/2

 if y > floorY then
  y = floorY
  velocity = velocity * bounce
  if math.abs(velocity) < 50 then
   velocity = 0
   onGround = true
  end
 end

 ball.Position = UDim2.new(0, x, 0, y)
end)
