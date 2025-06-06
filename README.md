local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Create ScreenGui
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ScriptExecutorGUI"
ScreenGui.Parent = game:GetService("CoreGui")
ScreenGui.ResetOnSpawn = false

-- Main Frame
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 400, 0, 300)
MainFrame.Position = UDim2.new(0.5, -200, 0.5, -150)
MainFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

-- Title Bar
local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Size = UDim2.new(1, 0, 0, 30)
TitleBar.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Name = "TitleLabel"
TitleLabel.Size = UDim2.new(1, -60, 1, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "Script Executor"
TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleLabel.TextSize = 18
TitleLabel.Font = Enum.Font.SourceSansBold
TitleLabel.Parent = TitleBar

local CloseButton = Instance.new("TextButton")
CloseButton.Name = "CloseButton"
CloseButton.Size = UDim2.new(0, 30, 0, 30)
CloseButton.Position = UDim2.new(1, -30, 0, 0)
CloseButton.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
CloseButton.Text = "X"
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.TextSize = 18
CloseButton.Font = Enum.Font.SourceSansBold
CloseButton.Parent = TitleBar

-- Script Box (Multi-line support)
local ScriptBox = Instance.new("TextBox")
ScriptBox.Name = "ScriptBox"
ScriptBox.Size = UDim2.new(1, -20, 1, -80)
ScriptBox.Position = UDim2.new(0, 10, 0, 40)
ScriptBox.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
ScriptBox.BorderSizePixel = 0
ScriptBox.TextColor3 = Color3.fromRGB(255, 255, 255)
ScriptBox.TextSize = 14
ScriptBox.Font = Enum.Font.Code
ScriptBox.TextXAlignment = Enum.TextXAlignment.Left
ScriptBox.TextYAlignment = Enum.TextYAlignment.Top
ScriptBox.ClearTextOnFocus = false
ScriptBox.MultiLine = true  -- Enable multi-line input
ScriptBox.PlaceholderText = "Enter your script here..."
ScriptBox.Parent = MainFrame

-- Error Display (for script errors)
local ErrorDisplay = Instance.new("TextLabel")
ErrorDisplay.Name = "ErrorDisplay"
ErrorDisplay.Size = UDim2.new(1, -20, 0, 40)
ErrorDisplay.Position = UDim2.new(0, 10, 1, -80)
ErrorDisplay.BackgroundTransparency = 1
ErrorDisplay.TextColor3 = Color3.fromRGB(255, 0, 0)
ErrorDisplay.TextSize = 14
ErrorDisplay.Font = Enum.Font.SourceSans
ErrorDisplay.Text = ""
ErrorDisplay.TextWrapped = true
ErrorDisplay.Parent = MainFrame

-- Execute Button
local ExecuteButton = Instance.new("TextButton")
ExecuteButton.Name = "ExecuteButton"
ExecuteButton.Size = UDim2.new(0, 100, 0, 30)
ExecuteButton.Position = UDim2.new(0, 10, 1, -40)
ExecuteButton.BackgroundColor3 = Color3.fromRGB(0, 120, 0)
ExecuteButton.Text = "Execute"
ExecuteButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ExecuteButton.TextSize = 16
ExecuteButton.Font = Enum.Font.SourceSansBold
ExecuteButton.Parent = MainFrame

-- Clear Button
local ClearButton = Instance.new("TextButton")
ClearButton.Name = "ClearButton"
ClearButton.Size = UDim2.new(0, 100, 0, 30)
ClearButton.Position = UDim2.new(0, 120, 1, -40)
ClearButton.BackgroundColor3 = Color3.fromRGB(120, 120, 0)
ClearButton.Text = "Clear"
ClearButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ClearButton.TextSize = 16
ClearButton.Font = Enum.Font.SourceSansBold
ClearButton.Parent = MainFrame

-- Add New Line Button (Green Button)
local AddLineButton = Instance.new("TextButton")
AddLineButton.Name = "AddLineButton"
AddLineButton.Size = UDim2.new(0, 100, 0, 30)
AddLineButton.Position = UDim2.new(0, 230, 1, -40)  -- Place it next to Clear button
AddLineButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)  -- Green color
AddLineButton.Text = "Add Line"
AddLineButton.TextColor3 = Color3.fromRGB(255, 255, 255)
AddLineButton.TextSize = 16
AddLineButton.Font = Enum.Font.SourceSansBold
AddLineButton.Parent = MainFrame

-- Line counter
local lineCounter = 1

-- Functionality for dragging
local dragging
local dragInput
local dragStart
local startPos

local function updateInput(input)
    local delta = input.Position - dragStart
    MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

TitleBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        updateInput(input)
    end
end)

CloseButton.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

-- Execute Button Functionality
ExecuteButton.MouseButton1Click:Connect(function()
    local script = ScriptBox.Text
    if script and script ~= "" then
        local success, err = pcall(function()
            loadstring(script)()
        end)
        if success then
            ErrorDisplay.Text = "Script executed successfully."
            ErrorDisplay.TextColor3 = Color3.fromRGB(0, 255, 0)  -- Green for success
        else
            ErrorDisplay.Text = "Error: " .. tostring(err)
            ErrorDisplay.TextColor3 = Color3.fromRGB(255, 0, 0)  -- Red for error
        end
    else
        ErrorDisplay.Text = "No script to execute."
        ErrorDisplay.TextColor3 = Color3.fromRGB(255, 165, 0)  -- Orange for warning
    end
end)

-- Clear Button Functionality
ClearButton.MouseButton1Click:Connect(function()
    ScriptBox.Text = ""
    ErrorDisplay.Text = ""
    lineCounter = 1  -- Reset line counter when clearing the box
end)

-- Add Line Button Functionality
AddLineButton.MouseButton1Click:Connect(function()
    local currentText = ScriptBox.Text
    ScriptBox.Text = currentText .. "\n-- Line " .. lineCounter  -- Add a new line with a comment
    lineCounter = lineCounter + 1  -- Increment line counter after adding a line
end)

-- Initial Setup
ErrorDisplay.Text = "Script Executor successfully loaded."
ErrorDisplay.TextColor3 = Color3.fromRGB(0, 255, 255)  -- Cyan for initial info
