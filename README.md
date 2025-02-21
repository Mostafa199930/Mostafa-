using System;
using System.Text.RegularExpressions;
using MinecraftClient;

class DragonRouletteBot : ChatBot
{
    private string lastPayer = "";
    private long lastBetAmount = 0;

    public override void GetText(string text)
    {
        // Detect payment messages (assuming chat shows them)
        Match match = Regex.Match(text, @"(\w+) paid you (\d+)[B|T]");
        if (match.Success)
        {
            lastPayer = match.Groups[1].Value;
            string amountStr = match.Groups[2].Value;
            lastBetAmount = ConvertToNumber(amountStr);
            
            if (lastBetAmount < 500_000_000_000)
            {
                SendText($"/msg {lastPayer} Minimum Bet 500B");
            }
            else if (lastBetAmount > 15_000_000_000_000)
            {
                SendText($"/msg {lastPayer} Maximum Bet 15T");
            }
            else
            {
                LogToConsole($"Bet received from {lastPayer}: {lastBetAmount}");
                SendText($"Rolling <DG> For {lastPayer} ....");
                BreakDragonEgg();
            }
        }
    }

    private long ConvertToNumber(string amountStr)
    {
        if (amountStr.EndsWith("B")) return long.Parse(amountStr.TrimEnd('B')) * 1_000_000_000;
        if (amountStr.EndsWith("T")) return long.Parse(amountStr.TrimEnd('T')) * 1_000_000_000_000;
        return long.Parse(amountStr);
    }

    private void BreakDragonEgg()
    {
        LogToConsole("Breaking dragon egg...");
        PerformInternalCommand("use"); // Simulates breaking the egg
        Wait(2000);
        CheckBlockUnderEgg();
    }

    private void CheckBlockUnderEgg()
    {
        // Simulated block detection (Replace with actual detection if server allows it)
        string blockType = SimulateBlockDetection(); 

        if (blockType == "green")
        {
            PayPlayer(lastPayer, lastBetAmount * 2);
            SendText($"/msg {lastPayer} You have won the <DG> Of {lastBetAmount}");
        }
        else if (blockType == "purple")
        {
            PayPlayer(lastPayer, lastBetAmount * 3);
            SendText($"/msg {lastPayer} You have won the <DG> Of {lastBetAmount}");
        }
        else
        {
            LogToConsole($"{lastPayer} won nothing.");
            SendText($"/msg {lastPayer} You Have Lost The <DG> Of {lastBetAmount}");
        }
    }

    private void PayPlayer(string player, long amount)
    {
        SendText($"/pay {player} {amount}");
        LogToConsole($"Paid {player} {amount}");
    }

    private string SimulateBlockDetection()
    {
        // This is just a placeholder, since MCC can't check blocks directly
        string[] blocks = { "green", "purple", "black" };
        Random rand = new Random();
        return blocks[rand.Next(blocks.Length)];
    }
}
