-- Orion carregamento
local OrionLib = loadstring(game:HttpGet("https://raw.githubusercontent.com/jensonhirst/Orion/main/source"))()
local Window = OrionLib:MakeWindow({
Name = "RedBlack - Fling Bus",
HidePremium = false,
SaveConfig = true,
IntroText = "Fling Bus Ativado",
ConfigFolder = "RedBlackHub"
})

-- Serviços
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RE = game:GetService("ReplicatedStorage"):WaitForChild("RE")
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")
local PosInicial = HumanoidRootPart.Position
local SelectedPlayer = nil

-- Função: Teleporta pro spawn, spawna o ônibus e senta no banco do motorista
local function SpawnESentar()
Character:MoveTo(Vector3.new(608.35, 5.34, 265.23))
task.wait(1)

-- Spawna o ônibus  
RE["1Ca1r"]:FireServer("PickingCar", "SchoolBus")  

-- Espera o ônibus aparecer  
local bus  
for i = 1, 20 do  
    task.wait(0.2)  
    for _, v in ipairs(workspace.Vehicles:GetChildren()) do  
        if v:FindFirstChildWhichIsA("VehicleSeat", true) then  
            local dist = (v:GetPivot().Position - HumanoidRootPart.Position).Magnitude  
            if dist < 30 then  
                bus = v  
                break  
            end  
        end  
    end  
    if bus then break end  
end  

if not bus then return false end  

-- Acha o banco do motorista  
local driverSeat = nil  
for _, v in ipairs(bus:GetDescendants()) do  
    if v:IsA("VehicleSeat") then  
        driverSeat = v  
        break  
    end  
end  

if driverSeat then  
    for i = 1, 10 do  
        HumanoidRootPart.CFrame = driverSeat.CFrame + Vector3.new(0, 2, 0)  
        task.wait(0.2)  
        Character.Humanoid.Sit = true  
        task.wait(0.2)  
        if Character.Humanoid.SeatPart == driverSeat then  
            return true  
        end  
    end  
end  

return false

end

-- Fling Bus com verificação e retorno
local function FlingBusTarget(targetPlayer)
if not targetPlayer or not targetPlayer.Character or not targetPlayer.Character:FindFirstChild("HumanoidRootPart") then return end

local ok = SpawnESentar()  
if not ok then  
    OrionLib:MakeNotification({  
        Name = "Erro",  
        Content = "Não foi possível se sentar no ônibus!",  
        Image = "rbxassetid://4483345998",  
        Time = 5  
    })  
    return  
end  

local targetHumanoid = targetPlayer.Character:WaitForChild("Humanoid")  
local connection  

connection = targetHumanoid:GetPropertyChangedSignal("SeatPart"):Connect(function()  
    if targetHumanoid.SeatPart and targetHumanoid.SeatPart:IsDescendantOf(workspace.Vehicles) then  
        connection:Disconnect()  

        -- Void  
        local voidPos = Vector3.new(9999, -500, 9999)  
        HumanoidRootPart.CFrame = CFrame.new(voidPos)  
        task.wait(0.4)  

        RE["1Ca1r"]:FireServer("DeleteAllVehicles")  
        task.wait(0.4)  

        HumanoidRootPart.CFrame = CFrame.new(PosInicial)  

        OrionLib:MakeNotification({  
            Name = "Fling Realizado",  
            Content = "Jogador foi pro espaço!",  
            Image = "rbxassetid://4483345998",  
            Time = 4  
        })  
    end  
end)  

-- Gira em volta do jogador  
task.spawn(function()  
    while true do  
        if not targetPlayer or not targetPlayer.Character then break end  
        if targetPlayer.Character:FindFirstChild("Humanoid").SeatPart then break end  
        HumanoidRootPart.CFrame = targetPlayer.Character.HumanoidRootPart.CFrame * CFrame.new(math.random(-3, 3), 0, math.random(-3, 3))  
        task.wait(0.1)  
    end  
end)

end

-- Aba e controles
local TrollTab = Window:MakeTab({
Name = "Troll",
Icon = "rbxassetid://4483345998",
PremiumOnly = false
})

TrollTab:AddTextbox({
Name = "Nome do Jogador",
Default = "",
TextDisappear = false,
Callback = function(txt)
SelectedPlayer = txt
end
})

TrollTab:AddButton({
Name = "🎩Fling Bus🚋",
Callback = function()
local target = Players:FindFirstChild(SelectedPlayer)
if target then
FlingBusTarget(target)
else
OrionLib:MakeNotification({
Name = "Erro",
Content = "Jogador não encontrado!",
Image = "rbxassetid://4483345998",
Time = 4
})
end
end
})
